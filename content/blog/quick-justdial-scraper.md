+++
title = "Quick JustDial scraper"
date = "2015-09-14T00:00:00-00:00"
tags = ["data", "cheatsheet", "notes", "scraper", "python"]
type = "post"
+++

So my friend asked me to scrape data from JustDial and give it to him in an excel sheet.
I thought let's give it a try. He needed
name of firm, address and phone number of any JustDial URL he wants to scrape. After effectively
around 4 hours of work, the below script was created.

Note that the script is dirty. You need to edit the jd_url to search any other URL. Also,
the looping will go on forever, so you have to keep checking the file size of generated
'data.csv' file, and when you're sure it's not increasing any more, kill the script by
pressing CTRL+C. This script works as of today. Tomorrow it might not. Also, excuse
stray comments/bad formatting of code. I'm not sure I want to clean it right now :)

Feel free to use/modify it the way you want.


    # PIP requirements: requests, beautifulsoup4
    import requests
    from bs4 import BeautifulSoup
    import json
    import csv
    
    jd_url = "http://www.justdial.com/Bangalore/Car-Hire-%3Cnear%3E-Shanthinagar"
    
    # Split http/https prefix if any
    # TODO: work on URLs which dont' have the CT part in URL
    jd_url = jd_url.split('http://www.justdial.com/')[-1].split('https://www.justdial.com/')[-1]
    city, search, cat_id = '', '', ''
    split_vals = jd_url.split('/')
    if len(split_vals) == 3:
        city, search, cat_id = jd_url.split('/')
        cat_id = cat_id.split('-')[-1]
    elif len(split_vals) == 2:
        city, search = jd_url.split('/')
    search = search.replace('-', '+')
    
    
    with open('data.csv', 'w') as f:
        #writer = csv.writer(f, delimiter=',', quoting=csv.QUOTE_ALL, lineterminator='\n')
    
        page = 1
        while True:
            print 'page', page
            resp = requests.get('http://www.justdial.com'+'/functions/ajxsearch.php?national_search=0&act=pagination&city={0}&search={1}&where=&catid={2}&psearch=&prid=&page={3}'.format(city, search, cat_id, page))
            markup = resp.json()['markup'].replace('\/', '/')
            soup = BeautifulSoup(markup, 'html.parser')
    
    
            for thing in soup.find_all('section'):
                csv_list = []
                if thing.get('class')==[u'jcar']:
                    # Company name
                    for a_tag in thing.find_all('a'):
                        if a_tag.get('onclick')=="_ct('clntnm', 'lspg');":
                            csv_list.append(a_tag.get('title'))
    
                    # Address
                    for span_tag in thing.find_all('span'):
                        if span_tag.get('class')==[u'mrehover', u'dn']:
                            csv_list.append(span_tag.get_text().strip())
    
                    # Phone number
                    for a_tag in thing.find_all('a'):
                        if a_tag.get('href').startswith('tel:'):
                            csv_list.append(a_tag.get('href').split(':')[-1])
    
    
                    csv_list = ['"'+item+'"' for item in csv_list]
                    writeline = ','.join(csv_list)+'\n'
                    f.write(','.join(csv_list)+'\n')
            page+=1

Cheers!
