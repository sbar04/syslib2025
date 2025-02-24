# Library Search with yaz-client
This is where I have felt the most lost so far! Brain is stretching!

## What is the yaz-client???
* SRU (Search/Retrieve via URL) and SRW (Search/Retrieve Web service) client
* Standard protocols in libraries for sharing, querying, and retrieving bib info between library databases which is *super cool*
* Alma lets institutions be a SRU target with this base URL: The base URL for SRU requests is: https://<Alma domain>/view/sru/<institution code>

### Documentation
* `yaz` can search many bib attributes which can be found using `man bib1-attr`
* Less comprehensive overview from [LOC](https://www.loc.gov/z3950/agency/defns/bib1.html)

## Using yaz-client
The fun part!
I needed to open the library's OPAC using the server address
`Z> open saalck-uky.alma.exlibrisgroup.com:1921/01SAA_UKY`

Alma also has a Guest sandbox URL: https://na01.alma.exlibrisgroup.com/view/sru/API_GUEST_INST

### Using Queries
* Queries use Prefix Query Notation (PQN) which is where the Boolean operator preceds the search terms, attributes, fields
* `man yaz-client` has a COMMANDS section with all possible commands
* Queries go Command --> PQN --> search syntax articulated by PQN
* Example: `Z> find @and @attr 1=4 "information" @attr 1=21 "library science"`
`find` is the command; sends search request. Can be shorted to just `f`
`@and` is the operator; searching AND of the two following attributes
`@attr 1=4` is telling it to search for the term "information" in the title field
`@attr 1=21` is telling it to search for "library science" in the subject heading field

To see the results, need the `show` command

`show 1`
`show 2`

To append these to a file use the -m option when starting

`yaz-client -m records.marc`

```
Z> open saalck-uky.alma.exlibrisgroup.com:1921/01SAA_UKY
Z> find @and @attr 1=4 "information" @attr 1=21 "library science"
Z> show 1
Z> show 2
Z> show 3
Z> quit
```

Or show as many as you want using `show 1 +120` (or however many the query finds)


### Making that Readable
Now I have a .marc file which can be converted into friendlier formats
`yaz-marcdump -o json records.marc > records.json`

Then use `jq` command for better readability
`jq . records.json > records-formatted.json`

There are a lot of things to learn here to normalize the data and make everything more readable.
But, even just connecting via SRU is making me feel like this:

![Hacker Voice I'm in](Images/Hacking.jpg)

### Loose Goals
* Get more familiar with the Bib-1 attributes
* Play around with JSON sites like [JSON Formatter](https://jsonformatter.org/) to look at data structure like 
* Try exporting a set to XML file just to see the difference?
* Get more familiar with jq searching
