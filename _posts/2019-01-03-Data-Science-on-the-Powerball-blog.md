---
layout: post
title:  "Web Scraper to gather Historical Powerball Lottery Data"
date:   2019-01-05 21:46:04
categories: Machines

---
## Introduction
In the previous blog post, we explored the mathematical underpinnings behind the Powerball lottery. We learned that using enumerative combinatorics in the binomial coefficient, we can calculate the number of possible combinations that exist within any given Powerball drawing. Within this next article, we will learn how to build a web scraper using Python that gathers historical winning Powerball Number combinations. <!--more--> Once we have gathered the scraped information, we will learn how to clean it and convert it into it usable Pandas DataFrame.

## Brief Web Scraping Overview

A Web scraper is a program that uses the hypertext transfer protocol to request information from websites, and then saves that information for some purpose. A web scraper behaves similar to an API, in that it can be run to consistently return desired information from a website. More often than not the websites that house the information that we are interested in are not robust enough to have an API. Thus, we have to be clever as to how we extract information that we want.

A rudimentary example of a web scraper is to simply copy and paste the information that we’re looking for into a program like Google sheets or Microsoft Excel. Beyond simply being tedious and time intensive, there is another reason why copying, and pasting is not an optimal web scraping process. Say for instance the information that is being scraped changes periodically. The individual who is doing the copying and pasting would have to iteratively go back and copy and paste the new information each time a change occurred. In this article we will build and implement a web scraper using Python. It will allow us to iteratively and consistently collect the information that we’re looking for each time we use it.

### Legality of Web Scrapers

Using this point here in the article I do want to say that there is a matter of legality associated with web scrapers. One, the way that web scrapers are built they can iteratively make thousands of HTTP get requests from sites in a very short amount time. This when done improperly can send large unexpected traffic loads to server hosting website potentially cause the site to go down. Two, if the site explicitly states within their terms of service (ToS) or within their prohibited operations page, that automated data collection is not allowed, then don’t do it.

A great way to check the sites rules on web scraping is to look for a page called robots.txt. On most sites this file denotes what is and is not allowed for robots to crawl. This article we will be using the Wisconsin lottery’s website to collect historical Powerball winning ticket combinations. Going to [Wisconson Lottery’s Robots.txt File](https://www.wilottery.com/robots.txt), we see that there are only two disallowed processes for web scrapers. Luckily, we will be scraping neither of these sections of the site.

### How Web Scrapers Work

There are three primary components of any web scraper. The first component of any web scraper gives it the ability to parse the HTML of a given website. The second component searches over the HTML and gathers the information of interest. Within our web scraper we will define a function called *parse_and_gather()*  that does both of these processes automatically for us.

Once the web scraper has parsed the HTML and gathered the pertinent information, it is up to the developer to decide what the most applicable format to convert the data to. Thus, the final component have any web scraper formats together data into the appropriate format. In our case, we will write another function called *build_dataframe()*, that takes the gathered information and converts it into a Pandas DataFrame.


## Import the Necessary Packages

{% highlight python %}
import pandas as pd
from requests import get
import lxml.html as lh

{% endhighlight %}

[**Pandas**](https://pandas.pydata.org) - Pandas is an Open-Source Project and a python library that helps python users work with multidimensional data, or panel data in a computational form.

[**Requests**](http://docs.python-requests.org/en/master/) - Requests is a python library that makes using the Hypertext Transfer Protocol via python a simple to do.

[**Lxml**](https://lxml.de) - LXML is a python library that allow python users a way to computationally interact with HTML or XML elements.

## Build a Header dictionary


After importing the necessary Python libraries let us begin to discuss the hypertext transfer protocol at a little more depth. There’re two primary actors within HTTP process. There is the client, who requests information from the server. And then there is the server which responds to the client by sending resources like HTML files and other content. HTTP was built to facilitate and set standardized guidelines to this request and response process between a client and the server. Within a client request to a server there are typically three primary components that make up a request message. The first part you have the request line otherwise known as the client request method. To get request tells the server explicitly the information that the client is requesting to be served. They are several different kinds of request methods the client can make to a server, these include GET, POST, DELETE, TRACE, CONNECT, OPTIONS, PUT, and HEAD requests. The second part of the request message we call HTTP header fields which are used to relay specific information about the client to the server. In the final component is an optional message that can be sent to the specific server from the client.

When building a web scraper, you must have the first component of a client request message to gather the information from the server. The two most common request methods that we use within our web scrapers are the GET and POST methods. GET method allows us to request a specific set of information from the server, whereas the POST method allows us to send a specific set of information to server. Within our web scraper we will use an GET method to request the relevant Powerball data from the Wisconsin lottery website.

Though not a necessity is always good practice to include the second component of the client request method within your web scraper. Again, remember that the second component is the request a header. The reason why this is so crucial is because it allows the server to verify that the client requesting information is a valid client. HTTP request headers are also helpful in what’s known as content negotiation, where clients specify to the server the specific type of content that they’re requesting.

For our specific case we will use a python dictionary named headers to define our HTTP request headers.

{% highlight python%}
headers = {	'accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8',
            'accept-encoding': 'gzip, deflate, sdch, br',
            'accept-language': 'en-GB,en-US;q=0.8,en;q=0.6',
            'upgrade-insecure-requests': '1',
            'user-agent': 'Mozilla/5.0 (X11; Linux x86_64) Chrome/51.0.2704.79 Safari/537.36',
            'Cache-Control': 'no-cache',
            'Connection': 'keep-alive' }
{% endhighlight %}

We have an *accept* request field that tells the server the types of media that are acceptable to send back to us from the GET request. Here we define several different variants of HTML or XML that is appropriate to send it back. The addition of a lowercase ‘q’ in any request field indicates to the server something called the relative quality factor. This parameter is a range from [0-1] and defines to the relative degree of preference for each content type. If there is no q parameter defined, the entry is defaulted to a q parameter of 1. Further breaking down our accept request field, we see that the entries *’text/html’* and *’application/xhtml+xml’* do not have a q parameter, but the final entries *’application/xml;q=0.9’* and ,*’asterisk /asterisk’=0.8'* both do. This means that we are telling the server that we prefer the first to entries as they default to 1, and we prefer the final two entries at 90% and 80% respectively.{% sidenote 'One' 'The ‘asterisk /asterisk’ requests from the server to use all other forms of the entry.'%}

The next request field is the *accept-encoding* field, which defines the type of content compression that is acceptable for the server to send back to us. In our case, this request field might be slightly redundant, because the information or requesting should needs no compression. Yet, rather than taking it out I thought I’d leave it in for you to see. After we define the acceptable compression, we then define the acceptable language, if applicable, for the server to send back information in. Here we denote this field as the *accept-language* field. We tell the server to send the information back to us in English (United States or British).




## Build the Parse & Gather Function



{% highlight python%}
def parse_and_gather():
    '''This function makes the GET request & collects the data'''

    #HTTP GET Call Setup
    url = 'https://www.wilottery.com/lottogames/powerballhistoryOD.aspx'
    response = get(url,headers=headers)
    document = lh.fromstring(response.content)
    table_elements = document.xpath('//tr')

    #Define the Column Headers for the 5 Empty Column Headers
    num_list = ['Num_1', 'Num_2', 'Num_3', 'Num_4', 'Num_5']

    #Fill in the Column Headers
    col_header = [i.text_content() for i in table_elements[0]]
    col = [[i,[]] for i in col_header]

    #Fill in the middle 5 empty column headers with num_list
    for i in range(0,len(num_list)):
        j = i+1
        col[j][0] = num_list[i]

    #Beacause the first row is the headers, the data is stored on the second row onwards
    for j in range(1,len(table_elements)):
        T=table_elements[j]

        # If the table has more than the 8 Columns on the WI Lottery Site
        if len(T)!=8:
            break

        #Convert the elements into integers and save them to col lists.     
        i=0
        for t in T.iterchildren():
            data=t.text_content()
            if i>0:
                try:
                    data=int(data)
                except:
                    pass
            col[i][1].append(data)
            i+=1
    return col
{% endhighlight %}


## Build the Formatting Function

{% highlight python%}

def build_dataframe():
    ''' This function creates a Pandas DataFrame from the data'''

    #Call the Gather Data Function
    col = parse_and_gather()

    #Convert to a Dictionary
    Dict={header:column for (header,column) in col}

    #Create a DataFrame
    df=pd.DataFrame.from_dict(Dict)

    #Set the Date Column as the index
    df.set_index('Draw Date', inplace=True)

    #Convert the Index to a Pandas DatetimeIndex
    df.index = pd.to_datetime(df.index)

    #Selects all Non-Digits
    digits = r"\D+"
    #Selects all spaces
    spaces = r"\s+"

    #Splits the powerplay column into just the integer values
    df['Power Play_Clean'] = df['Power Play'].astype(str).str.split(spaces).str.get(1).str.split(digits).str.get(0)

    #Drop the old Powerplay column
    df.drop('Power Play', axis=1, inplace=True)

    #Add a new Powerplay column that contains the cleaned information
    df['Power Play'] = pd.to_numeric(df['Power Play_Clean'], errors='coerce', downcast='integer')

    #Drop the Cleaning Column
    df.drop('Power Play_Clean', axis=1, inplace=True)

    return df

{% endhighlight %}
## View Output

## Conclusion
