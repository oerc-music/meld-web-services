MELD Web Services
=================

MELD Web Services are a collection of services that may be run in support of MELD applications, depending on their requirements. Historically, this has been fulfilled by the custom Flask/Python implementation of Session and Annotation Services forming the majority of the code in this repository.

Standard LDP servers provide an alternative to this custom implementation, in conjunction to the annotation templates within this repository. We have tested MELD with the [GOLD Linked Data Platform server](https://github.com/linkeddata/gold).

Please refer to the notes below on both our custom implementation, and on setting up a GOLD server.

For an overview of MELD, please see: [oerc-music/meld](http://github.com/oerc-music/meld).

Custom implementation of Session and Annotation Services
========================================================

To install:
-----------
```
git clone git@github.com:oerc-music/meld-web-services
cd meld-web-services
pip install -r requirements.txt #(or use a virtualenv)
source set_env.sh 
python manage.py runserver #(default port: 5000)
```

If you run into missing header files while trying the pip install step, you may first need to install the following packages using your OS package manager:

* libxml2-dev
* libxslt1-dev

This code depends on a number of Python2 modules, including Flask (web server), and PyLD, rdflib, and SPARQLWrapper (Linked Data functionalities around RDF graph handling, JSON-LD conversion, and SPARQL querying). A full listing of the dependencies is available in the requirements.txt file. 


Creating a session                                                     
------------------                                                     
                                                                       
To start a new session use the MELD session service as follows:        
                                                                       
curl -H "Content-Type: application/json" -H "Slug: SessionName" -d '{  
"@type": ["mo:Performance", "ldp:BasicContainer"], "mo:performance_of":
{ "@id": $SCORE_URI } }' -v http://127.0.0.1:5000/sessions             
                                                                       
n.b., the Slug is optional (it just expresses a preference for the URI 
of the resulting session).                                             
                                                                       
The $SCORE_URI corresponds to the conceptual score that the session is 
presenting (mo:performance_of).                                        
                                                                       
After the POST you should get back a response with status 201 and a    
"Location" header telling you the $SESSION_URI of the new session.     
                                                                       
You can then load the session in the jam client by going to:           
                                                                       
http://127.0.0.1:8080/Jam?session=$SESSION_URI                         
                                                                       
                                                                       
Posting an annotation                                                  
---------------------                                                  
                                                                       
Annotations can be posted through the client user interface, or directly to the session LDP container (Annotation Service). You will need to                                
add a "Content-Type" header with value "application/json" for the      
server to process them properly. To avoid race conditions and          
accidental overwriting, we use ETags (file hashes) which  need to be   
supplied with each POST. The sequence is as follows:                   
                                                                       
1. Perform a GET on the session URI, read the ETag header value from   
the response                                                           
2. POST the annotation to the session, supplying the same ETag value as
 the "If-None-Match" header                                            
3. Check the response status. If it's 201 (CREATED), we're done. If    
it's 412 (PRECONDITION FAILED), the file changed before our POST got   
through - so repeat from step 1.    


Running a standard LDP server (GOLD)
====================================
We have successfully tested [GOLD Linked Data Platform server](https://github.com/linkeddata/gold) as an off-the-shelf alternative to the custom implementation above. Here are our notes on getting GOLD to run:

## Install GOLD

Get dependencies Ubuntu:

    sudo apt-get install golang-go libraptor2-dev libmagic-dev

CentOS:

    sudo yum install golang raptor2-devel file-devel

Mac seems to be (untested): `brew install go raptor libmagic`

Setup paths and check go version:

    mkdir ~/go
    export GOPATH=~/go
    go version

Reported to need version >= 1.4

Install with go get:

    go get github.com/linkeddata/gold/server

## Run server

Make a data dir; copy config to server dir then edit to point to data dir:

    mkdir PATH-TO-DATA-DIR/gold-data/
    cp gold.conf $GOPATH/src/github.com/linkeddata/gold/server/gold.conf
    vi $GOPATH/src/github.com/linkeddata/gold/server/gold.conf

Start the server:

    $GOPATH/bin/server -conf=$GOPATH/src/github.com/linkeddata/gold/server/gold.conf
