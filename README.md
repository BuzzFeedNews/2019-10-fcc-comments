# Analysis of comments submitted to three FCC public dockets

This repository contains data, code, and methodology supporting [BuzzFeed News' analysis of comments submitted to three Federal Communications Commission (FCC) dockets](https://www.buzzfeednews.com/article/jsvine/net-neutrality-fcc-fake-comments-impersonation), published October 3, 2019:

- 17-108 ("Restoring Internet Freedom")
- 16-42 ("Expanding Consumers' Video Navigation Choices")
- 14-28 ("Protecting and Promoting the Open Internet")

Please see below for further details.

## Data Sources

The data in this repository comes from several sources:

### The FCC's Electronic Comment Filing System (ECFS)

The ECFS is the FCC's public portal for searching and accessing comments submitted to the commission's dockets. BuzzFeed News used the website to download each individually-listed comment, for two of the dockets: [14-28](https://www.fcc.gov/ecfs/search/filings?date_disseminated=%5Bgte%5D2014-02-21%5Blte%5D2016-01-01&proceedings_name=14-28&sort=date_disseminated,ASC_description=COMMENT) and [16-42](https://www.fcc.gov/ecfs/search/filings?date_disseminated=%5Bgte%5D2016-02-23%5Blte%5D2018-10-01&proceedings_name=16-42&sort=date_disseminated,ASC&submissiontype_description=COMMENT). __Note__: Not all comments submitted to the FCC are individually listed; in some cases, an organization will submit a consolidated set of comments as a PDF, with signatures and/or commenters' information listed in that PDF. Because of the extraordinary variety and inconsistency of those files, BuzzFeed News did not disaggregate those comments.

### The FCC's bulk download of Docket 17-108 comments

On November 7, 2017, [the FCC released](https://ecfsapi.fcc.gov/file/11073095518421/DA-17-1089A1_Rcd.pdf) a "complete set of [Docket 17-108] filings submitted as of November 3, 2017"; BuzzFeed News used this download to examine docket-wide trends.

### Bulk uploads to Docket 17-108, via FOIA

In response to two FOIA requests, the FCC provided to BuzzFeed News the files submitted to the agency's [bulk-upload system for Docket 17-108](https://www.fcc.gov/restoring-internet-freedom-comments-wc-docket-no-17-108), plus associated metadata indicating the uploader's Box.com account and the time of the upload. According to the FCC, it provided all such files submitted. Although the agency provided a template for the uploads, some of the files — typically the smallest ones, containing just one comment each — do not conform to them and could not be incorporated easily. Those comments, which represent an exceedingly small percentage of all bulk-uploaded comments, have not been included in this repository's data; in many cases, the corresponding comments appear also not to have been added to the FCC's public comment portal. In certain other cases, the upload files use non-standard column names. In cases where the intention appeared to be clear, BuzzFeed News fixed the column names and included the data.

### haveibeenpwned.com

[Have I Been Pwned](https://haveibeenpwned.com/) is a website and service that identifies whether any given email address has been exposed in any of hundreds of major data breaches. BuzzFeed News used [HIBP's application programming interface](https://haveibeenpwned.com/API/v3) to determine the most common breaches associated with various groups of email addresses.

## Personal Information Minimization

Because it appears that many of the comments in the data above were submitted without the consent of the named commenters, we have taken the following steps:

- Removing all raw personal-information columns (name, physical address, etc.).

- Replacing each distinct email address with a randomly-assigned unique identifier. (Specifically, a [version 4 UUID](https://www.cryptosys.net/pki/uuid-rfc4122.html).)

- Replacing each distinct email domain with a similar randomly-assigned unique identifier, except for very common domains. (Specifically the 36 domains that are associated with 10,000 or more unique email addresses in the Docket 17-108 comments.)

- Replacing each distinct combination of name + location (first line of street address, city, state, ZIP code) with another UUID. Before converting to UUIDs, ZIP codes are converted to zero-padded five-digit representations, and all strings are lowercased. For instance: `John Doe, 123 Smith Street, New York, NY 01111` will receive the same UUID as `john doe, 123 SMITH STREET, New York, ny 1111`, but neither will match submissions that put him at `123 Smith St.` (with the abbreviation).

## Data Files

The process above produces the files listed below. Several are too large to host on GitHub, so BuzzFeed News has [uploaded them here](https://archive.org/details/fcc-comments-and-bulk-uploads).

### Comment data

These files contain selected fields from the comment data listed above:

- `bulk-uploads-17-108-with-uuids.csv`: Docket 17-108 bulk uploads, via FOIA
- `comments-17-108-with-uuids.csv`: Docket 17-108, via FCC official download
- `comments-14-28-with-uuids.csv`: Docket 14-28, via FCC online portal
- `comments-16-42-with-uuids.csv`: Docket 16-42, via FCC online portal

They contain the following columns:

- `date`: The date of submission.
- `id_submission`: The ID the FCC has assigned to the comment. __Note__: Not available in `bulk-uploads-17-108-with-uuids.csv`, because the FCC assigns the IDs *after* they are uploaded.
- `comments`: The text of the comment. __Note__: This is sometimes modified by the FCC, for example by adding a filename or, as appears to be the case for some Docket 14-28 comments, removing boilerplate language.) __Note__: Not included in `comments-17-108-with-uuids.csv` for file-size considerations, because this file is mainly used for domain-counts.
- `name_and_location`: The UUID (see above) corresponding to the name and adress information provided with the comment. __Note__: Not included in `comments-17-108-with-uuids.csv`.
- `email_address`: The UUID (see above) corresponding to the email address provided with the comment. __Note__: In the FCC's commenting system, you don't have to control an email address to list it as the author of a comment.
- `email_address_nonstandard`: If the email address contains nonstandard characters (such as `%`) or formatting (such as lacking an `@` symbol), this value will be `1`; otherwise, it will be `0`. This is used to filter out likely-invalid addresses before checking them on Have I Been Pwned.
- `email_domain`: The domain of the email address, as a UUID unless it is one of the 36 domains described above.

Additionally, `bulk-uploads-17-108-with-uuids.csv` contains the following columns:

- `file`: The name of the file in which the comment was uploaded.
- `uploader`: The email address associated with the Box.com account that uploaded the file.

### Breach data

These files list the breaches, per Have I Been Pwned, for email addresses in a randomized samples of the comments bulk-uplaoded to Docket 17-108:

- `breaches-17-108-bulk-uploads-sample.csv`: 1,000-address sample of each of the eight bulk-uploaders whose Docket 17-108 uploads contained at least 10,000 unique email addresses.
- `breaches-17-108-mb-sample.csv`: 10,000-address sample of Media Bridge's Docket 17-108 bulk-uploads.


They contain the following columns:

- `email_address`: The UUID (see above) corresponding to the email address examined.
- `breach`: The name of the breach, [as returned by Have I Been Pwned](https://haveibeenpwned.com/API/v3).

## Analysis

The [`analyze-fcc-comments` notebook](notebooks/analyze-fcc-comments.ipynb) examines comments submitted to the three FCC dockets described above, the language used in them, the timing of their submission. For Docket 17-108, the notebook also examines the email domains associated with the comments, as well as rates at which the email addresses in the bulk uploads overlap with those exposed in major data breaches. The notebook also examines the overlap between the contact information in Docket 16-42 and Docket 17-108.

The [`analyze-mb-comment-structure` notebook](notebooks/analyze-mb-comment-structure.ipynb) examines the phrasing of the comments that Media Bridge submitted to Docket 17-108, and attempts to reverse-engineer the comments that use randomly-generated text.

## Reproducibility

The code running the analysis is written in Python 3, and requires the following Python libraries:

- [jupyter](https://jupyter.org/) to run the notebook infrastructure
- [pandas](https://pandas.pydata.org/) for data loading and analysis

If you would like to reuse the code for fetching data from Have I Been Pwned's API, you will also need these Python libraries:

- [requests](https://2.python-requests.org/en/master/) for HTTP requests
- [requests-cache](https://requests-cache.readthedocs.io/en/latest/) for caching HTTP requests
- [tqdm](https://tqdm.github.io) for progress bars

If you use Pipenv, you can install all required libraries with `pipenv install`.

As noted above, you will need to download the source data separately. Save the folder as this repository's `/data` directory.

Execute the notebooks in the `notebooks/` directory to reproduce the findings.

## Licensing

All code in this repository is available under the [MIT License](https://opensource.org/licenses/MIT).

## Questions / Feedback

Contact Jeremy Singer-Vine at [jeremy.singer-vine@buzzfeed.com](mailto:jeremy.singer-vine@buzzfeed.com).

Looking for more from BuzzFeed News? [Click here for a list of our open-sourced projects, data, and code.](https://github.com/BuzzFeedNews/everything)
