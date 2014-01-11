So I am currently on a clinical trial and we have a very interesting export process.  Essentially, the data comes in an XML format, then is converted to data sets using SAS.  That's all fine and good, because SAS is good at converting things.  The problem is, the people who started writing code to process the data and the majority of people maintaining the code use Stata.

Now that's a problem.  OK, well let's take these SAS data sets and convert them to Stata using Stat-Transfer.  Not totally reprehensible, it at least preserves some form versus a CSV to dta (Stata format) nightmare.  The only problem with this is that for some reason, the SAS parsing of the XML started to chew up a bit of memory, for about 140 data sets (it's a lot of forms).  Oh, by the way, a bit of memory was about *16 gigs* from a *100 meg* file. That's atrocious.  I don't care what it's doing but an 160 fold increase in just converting some XML and copying the datasets.  Not only that, it took *over 4 hours* What the hell SAS?

Anyway, we just started a phase III trial with similar data from before. We're using the same database.  I decided to stop the insanity and convert the XML in `R`.  The data still needs to produce in Stata data sets, but at least I could likely control the memory consumption and the time limits.  At least I could throw it onto our computing cluster if things got out of control (I guess I could have done that with SAS, but come on.).  

Now, thank God that the `XML` package exists.  So pretty much just using some `xmlParse` and `xmlToDataFrame` commands, I had some `data.frame`s! Now, just make them into Stata data sets right? Wrong.  

Let me first say that the `foreign` package is awesome.  You can read in pretty much any respectable statistical software dataset into `R` and do some exporting.  Also, the `SASxport` package allows you to export to the XPORT format using `write.xport`, which is a widely used (and FDA-compliant) format.

Now what problems did I have with the `foreign` package?

* I believe that Stata data sets can have length-32 variable names.  After some correspondence, the maintainers argue that Stata's documentation only support "up to 32" characters, which means only 31.  The [documentation](http://www.stata.com/help.cgi?dta_113) states
> varlist contains the names of the Stata variables 1, ..., nvar, each up 
> to 32 characters in length

A week after my discussion, `foreign` had noted in their [ChangeLog](http://cran.r-project.org/web/packages/foreign/ChangeLog):

>  man/{read,write}.dta: Freeze Stata support.

Well, that's a shame.  I guess I'll just change `write.dta` to do what I want.

* Empty strings in `R` are represented as "".  According to the `foreign` package, Stata documentation states that empty strings is not supported, which is true:
> 1.  Strings in Stata may be from 1 to 244 bytes long.

and "" has 0 bytes:

```r
nchar("", "bytes")
```

```
## [1] 0
```



```bash
/Applications/Stata/Stata.app/Contents/MacOS/stata "test.do"
```



```bash
cat test.log
```

```
## --------------------------------------------------------------------------------------------------------
##       name:  <unnamed>
##        log:  /Users/muschellij2/Dropbox/Public/WordPress_Hopstat/XML_to_Stata/test.log
##   log type:  text
##  opened on:  11 Jan 2014, 12:55:20
## 
## . set obs 1
## obs was 0, now 1
## 
## . gen x = ""
## (1 missing value generated)
## 
## . count if mi(x)
##     1
## 
## . count if x == ""
##     1
## 
## . log close
##       name:  <unnamed>
##        log:  /Users/muschellij2/Dropbox/Public/WordPress_Hopstat/XML_to_Stata/test.log
##   log type:  text
##  closed on:  11 Jan 2014, 12:55:20
## --------------------------------------------------------------------------------------------------------
```



I know from reading in Stata dta files, character variables can have data "", it's treated as missing.  his isn't a major problem, as long as you know about it.  
