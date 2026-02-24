+++
date = '2019-03-11T17:05:14-05:00'
draft = false
title = 'Down in the muck of data'
+++

I've been back in the ULP dataset. I was working in it about three weeks ago and set it aside while I went down to Boston to work on other projects. I had some spare time this weekend, though, and decided to open it back up.

I'd foundered on cleaning up and encoding the counties that I talked about in [this post][0]. Both state and county names were saved using FIPS codes, but the data has plenty of typos. This shouldn't surprise us. All of these records were typed in by a person, and there are more than 440,000 of them. Even a 99.5% accuracy rate would leave more than 2,000 corrupted entries. Thus cleaning, which always takes the same dreary form:

- Tabulate the recorded values
- Flag any values that shouldn't appear, given your list of acceptable values
- Try to resolve what the anomalous value should have been, if possible
- Set any remaining anomalous values to missing
- Encode the sanitized list of characters into a factor variable

This is simplest with states, which are only supposed to have around 50 values. You know that `37` is a legitimate value because it's a valid FIPS code for a state (North Carolina, FWIW). By contrast, `&7` is an error, because there is no FIPS code with an ampersand. And `3` is ambiguous--it could be `03`, `30`, or something else entirely. Finally, a number like `81` is outside the range of FIPS values. In all of these cases, I would set the state value of those records to missing. So, I tablute the state variable, find all of the typos, and do that:

```R
stateStrikes <- c('  ', '& ', '&7', '0 ', '00', '0D', '2Y', '3 ', '4 ', '52',
                  '53', '54', '55', '5K', '61', '67', '70', '71', '72', '73',
                  '74', '79', '81', '87', '91', '92', '97', '99')
for (i in stateStrikes) {
    ulp <- ulp %>% mutate(state = na_if(state, i))
}
```

Ditto for counties. There're a lot more counties, and county is a three-character string, so there's more room for error. But for many types of typo, at least, the process is the same:

```R
countyStrikes <- c('   ', '  3', ' 09', ' 25', ' 27', ' 57', '-04', '-71',
                   '..2', '{35', '{39', '{79', '{EG', '{GE', '}00', '&  ',
                   '&&&', '0  ', '0 7', '0}1', '00 ', '00}', '00+', '000',
                   '01 ', '02}', '03 ', '04 ', '05 ', '05}', '06 ', '06}',
                   '08 ', '09 ', '0B9', '0X1', '1  ', '1 4', '1 5', '1?7',
                   '10-', '13 ', '13}', '15 ', '16 ', '16-', '17 ', '20 ',
                   '23}', '24}', '25}', '3  ', '3 1', '30{', '30}', '36{',
                   '37 ', '41 ', '7  ', '70}', '75}', '9  ', '90 ', '93}',
                   '96}', 'A13', 'A27', 'A45', 'C60', 'G50', 'J30', 'K40',
                   'K50', 'L40', 'Y11')
for (i in countyStrikes) {
    ulp <- ulp %>% mutate(county =
                              replace(county, county==i, NA))
}
```

It's more complicated for counties because the list of invalid county FIPS codes varies by state. Thus `007` is invalid in Delaware (which only has three counties) but fine in New Jersey. Most county FIPS codes are odd, but when you add a county after the fact (I'm looking at you, La Paz County, Arizona!) or have an independent city (Baltimore, St. Louis, too much of Virginia), they follow different schemes. There's no real way around this other than to tabulate the values *by state*, go through each state, and flag unacceptable ones. This is what a laptop and a lot of documentaries that you can listen to more than watch are for.

Eventually I created a named list of lists, then used a nested loop to visit each state and set its individual list of invalid values to missing. The savvy among you will recognize this as a dictionary bodged into R:

```R
countyNAs <- list(c('008', '066', '143', '145', '151', '155', '201', '295',
                 '300', '352', '510', '521', '745', '889'),
               c('006', '010', '020', '026', '030', '040', '046', '050',
                 '060', '063', '070', '077', '090', '110', '111', '113',
                 .
                 .
                 .
               c('040', '049', '079', '095', '141'))                 
names(countyNAs) <- c("Alabama", "Alaska", "Arizona", "Arkansas",
                      .
                      .
                      .
                      "West Virginia", "Wisconsin", "Wyoming")
for (i in names(countyNAs)) {
    for (j in countyNAs[[i]]) {
        ulp <- ulp %>% mutate(county = replace(county,
                                               state==i & county==j, NA))
    }
}
```

So far, so good.

# The limits of cleaning

Let's note right here how often judgment enters into this process. We don't talk about it enough. Most datasets have problems--typos, missing data, machine or language-encoding hiccups, shifts in coding schemes. The analyst has to make decisions about how to deal with these, and historically those decisions were almost never documented. To this day, no one wants to write up every single design choice they made. That is why it's important to share your data and code; but also that's why it's important to make lists like these, however tedious they seem. If I were to go through my data in Excel or the like, changing errors as I saw them, there would be no record of what I'd done, short of analyzing two datasets side by side. Even then, different rules could produce similar results in some cases. Don't just clean your data; whenever possible, clean your data with code.

Having *finally* cleaned up the counties and managed to encode them, I moved on to the `complainant` and `respondent` variables. Per the codebook, these are four-character strings:

muck1.png

Notice that there's no apparent translation for how to interpret those four characters! Elsewhere in this codebook they refer to another PDF that lists union names, of which I have a copy:

muck2.png

It seems that `036` corresponds to the Retail Clerks, per the codebook's example, so that's good. But what does the `7` at the start mean? And sure, I can imagine they use `000` to mean employer--or does that mean employer *association*, while `R` means employer? And what code indicates an individual?

Here, I'm running up against the limits of the available documentation. For my research, I mostly care whether it's the union or the employer filing the complaint, and I can probably back it out of these data, but it's worth noting that I *cannot* proceed from here without making some judgment calls. For the moment, I'm going to focus on the last three characters in these variables, and come back to the first ones.

# Suspicious error patterns

With that in mind, I tabulated the last three characters of the `complainant` variable, and started the same procedure as for states and counties. Eventually I compiled this list:

```R
complainantStrikes <- c("   ", "  -", "  }", "  &", "  1", "  2", "  3",
                        "  {", " { ", "{  ", " } ", "}  ", " & ", "&  ",
                        "  0", " 0 ", "0  ", "  1", " 1 ", " 3 ", "3  ",
                        "  4", " 4 ", "4  ", "0{ ", "0} ",
                        "  5", "  9", "  {", "  &", " 02", " 03", " 05",
                        " 07", " 21", " 22", " 23", " 25", " 26", "  3",
                        " 30", " 33", " 36", "  4", " 41", " 42", " 43",
                        " 46", " 50", " 51", " 52", " 63", " 71", " 72",
                        " 78", " 79", " 83", " 87", "---", ".23", "{00",
                        "&&&", "  0", " 0 ", " 1 ", "0 &", "0 0", "0 1",
                        "0 2", "0 3", "0 5", "0 6", " 0{", "0{&", "0{0",
                        " 0}", "0}5", " 00", "00 ", "00-", "00{", "00}",
                        "00&", "00C", "01 ", " 01", "01N", "01O", "01R",
                        "02 ", "02O", "03 ", "03A", "03J", "03O", "05 ",
                        "05M", " 06", "06 ", "06J", "06L", "07K", "07N",
                        "08 ", " 08", "0A5", "0A6", "0B3", "0C2", "0C6",
                        "0F1", " 0L", "1  ", "1 1", "1 3", "1 5", "1{3",
                        "1{5", " 10", "10 ", "10+", "10J", "10L", "10N",
                        " 11", "11 ", " 12", "12 ", "12J", "12N", " 13",
                        "13 ", "13}", " 14", "14 ", "14J", "14O", " 15",
                        "15 ", " 16", "16 ", "16O", " 17", "17 ", "17J",
                        "17K", "17N", "17P", "17Q", "17R", " 18", "18 ",
                        "18M", " 19", "19 ", "1D1", "1E1", "1H4", "2 1",
                        "2 3", "2 4", "2 9", "2{4", " 20", "20 ", "20F",
                        "20J", " 21", "21 ", "21+", "21A", "21C", "21J",
                        "21L", " 22", "22 ", "22K", " 23", "23 ", "23C",
                        "23L", " 24", "24 ", "24O", "25 ", " 25", " 29",
                        "29 ", "29J", "29K", "29P", "2A9", "2C3", "2D6",
                        " 3 ", "3  ", "3 1", "3 3", "3 8", "3 9", "3--",
                        " 30", "30 ", "30L", "31 ", " 31", "31L", " 32",
                        "32 ", " 33", "33 ", "34K", " 35", "35 ", "35*",
                        "35N", " 36", "36 ", "37J", "3B2", "3D2", "  4",
                        " 4 ", "4  ", "4 1", "4 2", " 40", "40 ", " 41",
                        "41 ", " 45", "45 ", "45J", " 46", "46 ", "46L",
                        " 47", "47 ", " 48", "48 ", "4F3", " 5 ", "5  ",
                        "5 3", "5 9", " 54", "54 ", " 56", "56 ", "5D1",
                        "5F3", "  6", " 6 ", "6  ", "6 1", "60L", " 61",
                        "61 ", " 62", "62 ", " 63", "63 ", " 68", "68 ",
                        " 71", "71 ", "72G", " 75", "75 ", " 76", "76 ",
                        "  8", " 8 ", "8  ", "8 1", "8 2", " 83", "83 ",
                        " 9 ", "9  ", "9 6", " 99", "99 ", "V00")
```

Yes, I'm aware how long that list is--*I had to type it*. But typing has its uses; you notice when you're hitting some keys more than others. For example, there are a lot of errors that include the first few letters of the alphabet. That might just be an encoding issue. But there are also a lot of errors that include keys from the right-hand side of the home row. In particular, the letters "J", "K", and "L" show up a lot more than you'd expect from random error, with "M", "N", "O", and "P" somewhere close behind.

That's *weird*, right? Erroneous "Q"s, "A"s, and "Z"s show up a lot, because they're right next to the Shift and Tab keys. "O" and "P" maybe, since they're below the "9" and "0" keys. But "J"? It's right in the middle of the letter keys.

Then it hit me: *these weren't typed on a modern keyboard*. These data come from punched cards, and those would have been typed on a specialized keypunch machine. I have *no* idea what those keyboards look like. To Google!

I have reason to suspect that the NLRB and AFL-CIO were using IBM mainframes. Even if I didn't, those were the most common type. With the rise of System/360, the most common keypunch machine was the IBM 29:

(By [waelder](https://de.wikipedia.org/wiki/Benutzer:waelder) - Own work, [CC BY 2.5](https://creativecommons.org/licenses/by/2.5), [Link](https://commons.wikimedia.org/w/index.php?curid=1962578))

If you squint, there are some telltale marks on that keyboard! Let's "enhance":

(By [Maximilian Dörrbecker](https://commons.wikimedia.org/wiki/User:Chumwa) - Own work, [CC BY-SA, 3.0](http://creativecommons.org/licenses/by-sa/3.0/), [Link](https://commons.wikimedia.org/w/index.php?curid=1588025))

Bingo. The IBM 29 didn't have a separate numeral row. Instead there was a telephone-style numeric keypad on the right-hand side of the keyboard. Since punched cards didn't use upper- and lowercase, you had "number-shift" and "letter-shift" keys (the "*numerisch*" and "*alpha*" keys here). If you stuttered on a "4", you'd enter a "J", and so on. That accounts for a lot of these typos. It even suggests where they ampersands might be coming from--a missed "3"!

# Delving deeper

This doesn't solve all problems, of course. It still isn't clear where all the "A"s, "B"s, and "C"s are coming from, nor what that first character in the complainant and respondent variables is supposed to represent. But whenever I find something like this--a connection between a seemingly random string of digits and the mechanical or human processes that generated it--I feel like I'm coming to understand my data better.

The next and biggest mystery, at least on this front, is where all the curly-brace characters are coming from! Bear in mind that, because these were punched cards from an old IBM mainframe, they were encoded in some variant of [EBCDIC](https://en.wikipedia.org/wiki/EBCDIC) instead of [ASCII](https://en.wikipedia.org/wiki/ASCII), and EBCDIC didn't *have* curly-brace characters. This suggests some sort of encoding-translation error, but parsing that may be a bridge too far, even for me.