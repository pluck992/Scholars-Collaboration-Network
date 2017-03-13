## Data Scrapping and Preparation Scripts*

#### 1.Find the real Awardee's Scopus ID.
*(Because the Awards for NSF Programs does not have the Scopus IDs for the awardees)*   

We need a file that contains **all the awards** and each award has its list of awardees. (This data set can be retrieved from [NSF Award Website][NSF])
  
Sample **JSON** format: **(Awards.json)**  
```json
[
    {
        "Award ID": "548391011"
        "Awardees": [
            "Awardee Name": "Kyunghyun, Cho",
            "Awardee Name": "Richard, Cole",
            "Awardee Name": "Ernest, Davis",
        ]
    },
    {
        "Award ID": "893417493"
        "Awardees": [
            "Awardee Name": "Rob, Fergus",
            "Awardee Name": "Lecun, Yann",
            "Awardee Name": "Davi, Geiger"
        ]
    }
]
```

We need a file *(or many files)* that contains **all the possible search results of a awardee's name** from Scopus website. For example, when searching for FirstName = Rob, LastName = Fergus, we may have many authors from Scopus that have the same/similar FirstName and LastName pairs.  

Sample single csv file format: **(Rob_Fergus_237.csv)**
```text
"Rob, Fergus", "34975053400", "University of Lucknow"
"Rob, F.",     "6602097199",  "Jawaharlal Nehru University"
"Rob Fergus",  "34769779500", "New York University"
"Rob Fergus",  "34769032443", "Stony Brook University State University of New York"
```

This file is retrieved by first using [this script][get_authors_html_page] to download the html file of a curl call:
```sh
#! /bin/bash

# This command grab the html website of a search result.
# if search for firstname = James, lastname = Kurose, we have a website specifically for all possible James Kurose
#nl ../../authorslist_urls.txt | xargs -n 2 -P 8 sh -c 'curl -b cookies_feb28.txt "$1" > author-$0.html'
let COUNTER=$2
while [ $COUNTER -lt 10 ]; read NAME
     do curl -b ../cookies/cookies_feb28.txt "$NAME" > author_$COUNTER.html
     sleep 0.5
     let COUNTER=COUNTER+1
done < $1
```

Then, use the [AwardeeInfoExtraction.ipynd][htmltocsv] to extract the author names, scopus IDs, and affiliations/universities, from the html file into a csv file, as the above sample *Rob_Fergus_237.csv* shown. 

Base on a Scopus ID, we can retrieve a list of co-authors from Scopus. *(We will scrap the data from the website to local just to improve time efficiency.)*  

Sample co-author file format: **(6602097199.txt)**
```text
Fei-Fei, Li
Freeman, William T.
Eigen, David
Lecun, Yann
Wan, Li
Weston, Jason L.
Baranec, Christoph J.
Davi, Geiger
Tran, Du V.
```

The counterpart file that contains more names from the list of awardees will be chosen. The Scopus ID will be assigned to the corresponding Awardee Name.  
This step may be done via **HDFS & MapReduce Framework** to improve time efficiency.

Eventually, we will have a **Awardee-Scopus-ID-Enabled-Award-List.json** structured file.

[img]

#### 2.Prepare the data clusters.  

- **url_scopus**: A file that contains the url that has access to a csv file contains all publication details of this author.
- **cookies.txt**: A file that contains the cookies needed to have access and download the csv.
'''bash
cat url_scopus | xargs curl -c cookies.txt -I
'''

'''bash
cat url_scopus | xargs curl -b cookies.txt > author.csv
'''

<sub><sup>**All the names and data are randomly chosen, does not mean anything related to the real scopus website or NSF programs. All data from Scopus and NSF program will NOT be published to anyone in this research.* </sup></sub>

[NSF]:https://www.research.gov/common/webapi/awardapisearch-v1.htm
[get_authors_html_page]:https://github.com/lizichen/collaboration_networks/blob/master/BashScript_AllPublications/all-authors-html/get_authors_html_page.sh
[htmltocsv]:https://github.com/lizichen/collaboration_networks/blob/master/BashScript_AllPublications/AwardeeInfoExtraction.ipynb