# web_scraping_elastic_search
I scraped to from  www.usom.gov.tr Turkish government web site that includes harmfull domain names and then i saved them at elasticsearch.

First of all i used urllib.request and BeautifulSoup libraries for scraping web site and with general purpose i wrote make_soup function that can request and parses html page.

def make_soup(url):
    page=urllib.request.urlopen(url)
    soupdata=soup(page,"html.parser")
    return soupdata

#Following to six line was wrote for elasticsearch. Elastics host default localhost/9200 and i create index which my scrape web sites name(usom), and web sites header information, at the header "AÇIKLAMA" means "definition", "KAYNAK" means "source and  "TARİH" means "date" in Turkish

ES_HOST = {"host" : "localhost", "port" : 9200}
INDEX_NAME = 'usom'
TYPE_NAME = 'zararli-baglantilar'
ID_FIELD = 'id'
bulk_data = []
header=["ID","URL","AÇIKLAMA","KAYNAK","TARİH"]

i also used datetime library because while i adding datas to elasticsearch i used datetime variable.

from datetime import datetime
i=0;date=[]
n=21 #Default i took 20 pages, but you can make it more or les.


#I used for loop for pagination and call soup function every time.
First if condition check if its first column its id, second if checks url and assignt dictionaries for elasticsearch, it goes on to fifth if condition, at the fifth cond. i took data as datetime.

for p in list(range(1,n)):
    string_version=str(p)
    soupd= make_soup("https://www.usom.gov.tr/zararli-baglantilar/"+string_version+".html")
    for record in soupd.findAll("tr"):
        data_dict = {}
        for rows in record.findAll("td"):
            if i%5==0:
                data_dict[header[i%5]]=rows.text
                idd=rows.text
            if i%5==1:
                data_dict[header[i%5]]=rows.text
            if i%5==2:
                data_dict[header[i%5]]=rows.text
            if i%5==3:
                 data_dict[header[i%5]]=rows.text
            if i%5==4:
                data_dict[header[i%5]]= datetime.strptime(rows.text, '%Y-%m-%d ')#i took date as date value.
            i=i+1
            #with the "i" value, my purpose was to control table values(td)
        #This op_dict also for Elasticsearch;
        op_dict = {
            "index": {
                "_index": INDEX_NAME,
                "_type": TYPE_NAME,
                "_id": idd
            }
        }
        bulk_data.append(op_dict)
        bulk_data.append(data_dict)
        
from elasticsearch import Elasticsearch
# create ES client, create index, if its name exist i deleted the previous one.
es = Elasticsearch(hosts=[ES_HOST])
if es.indices.exists(INDEX_NAME):
    print("deleting '%s' index..." % (INDEX_NAME))
    res = es.indices.delete(index=INDEX_NAME)
    print(" response: '%s'" % (res))

# since we are running locally, use one shard and no replicas
request_body = {
    "settings": {
        "number_of_shards": 1,
        "number_of_replicas": 0
    }
}
print("creating '%s' index..." % (INDEX_NAME))
res = es.indices.create(index=INDEX_NAME, body=request_body)
print(" response: '%s'" % (res))

# bulk index the data
print("bulk indexing...")
res = es.bulk(index=INDEX_NAME, body=bulk_data, refresh=True)
