# ElasticSearchNotes

ELASTICSEARCH NOTLAR

-Temel İşlemler
Bir sorgu aratmaya başlanıldığında 
query: ile başlıyoruz ilk olarak. Query sorgula anlamına geliyor.

bool: matchleşen kısımları almak için kullanılır. Queryden sonra gelir.
The default query for combining multiple leaf or compound query clauses, as must, should, must_not, or filter clauses. The must and should clauses have their scores combined — the more matching clauses, the better — while the must_not and filter clauses are executed in filter context.

filter: Burada filtremele işlemi yapılır, Array olarak açılır ve içerisinde obje olarak term verilir.
term: Bu koşulu temsil ediyor. If gibi düşünülebilir. Örnek sorgu aşağıdadır.

exists: Varlığını yokluğunu kontrol eder.
exists query 
Returns documents that contain any indexed value for a field.
"query": {
    "exists": {
      "field": "user"
    }
  }

ids: verileri idye göre bulmak için kullanılır.
Returns documents based on their IDs. This query uses document IDs stored in the _id field.
"query": {
    "ids" : {
      "values" : ["1", "4", "100"]
    }
  }

prefix: İçerisinde verilen veri geçen değerleri bulur.(Contains)
Returns documents that contain a specific prefix in a provided field.
 "query": {
    "prefix": {
      "user.id": {
        "value": "ki"
      }
    }
  }
term query: Verilen verinin aynısını field içinde arar.
Returns documents that contain an exact term in a provided field.
"query": {
    "term": {
      "user.id": {
        "value": "kimchy",
        "boost": 1.0
      }
    }
  }


terms query: Verilen birden fazla verinin aynısını field içinde arar.
Returns documents that contain one or more exact terms in a provided field.

"query": {
    "terms": {
      "user.id": [ "kimchy", "elkbee" ],
      "boost": 1.0
    }
  }

ids: documentIdye göre idleri verir.
ids query
Returns documents based on their document IDs.

range: aralıkları verir.
range query
Returns documents that contain terms within a provided range.
"query": {
    "range": {
      "age": {
        "gte": 10,
        "lte": 20,
        "boost": 2.0
      }
    }
  }

operator: And ve Or koşullarını sağlamak için araya konulur.
"operator":   "and" örneği

match: Direkt eşit olanı bulmak için kullanılır.
The match query is the standard query for performing a full-text search, including options for fuzzy matching.
MatchPhrase-MatchPhrasePrefix: Aranan kelime veya cümle geçmesi yeterli eşleşmeye bakmıyor.
minimum_should_match: minimum eşleşme sayısı verilir.
maximum_should_match: maximum eşleşme sayısını verir.
multi_match: Birden fazla match sorgusunu aynı anda üretmek için kullanılır.
The multi_match query builds on the match query to allow multi-field queries
"query": {
    "multi_match" : {
      "query":    "this is a test", 
      "fields": [ "subject", "message" ] 
    }
  }


must: Bir kurala uyması için kullanılır, array veya obje olabilir.
must_not: yukarının aynısı fakat olumsuz
bir objenin içine girilmek istenirse . ile ilerlenir.
"query": {
    "bool": {
      "filter": [
        { "term": { "color": "red"   }},
        { "term": { "brand": "gucci" }}
      ]
    }
  }
GET /shirts/_search
{
    "query":{
      "match":{
         "address":"mill"
      }
   }
}

-Paging işlemleri (queryden önce gelir.)
from: Es geçilecek data sayısı verilir.
size: Kaç tane alınacak, o bilgi verilir.
"from":4,
"size":5,
"query": {
    "bool": {
      "filter": [
        { "term": { "color": "red"   }},
        { "term": { "brand": "gucci" }}
      ]
    }
  }


-Nested işlemler(İç içe olan veriler!)
querynin altına açılır.
nested: obje olarak açılır ve arama yapılacak öğenin nested olduğu belirlenir.
Documents may contain fields of type nested. These fields are used to index arrays of objects, 
where each object can be queried (with the nested query) as an independent document.

path: Hangi objeye bakılması gerektiği seçilir.
query açılır altına ve matchlenecek key value değeri belirlenir.
inner_hits: Hangi değerlerin geldiği görülür.

"query": {
    "nested": {
      "path": "obj1",
      "query": {
        "bool": {
          "must": [
            { "match": { "obj1.name": "blue" } },
            { "range": { "obj1.count": { "gt": 5 } } }
          ]
        }
      },
      "score_mode": "avg"
    }
  }

 "query": {
    "nested": {
      "path": "comments",
      "query": {
        "match": {"comments.number" : 2}
      },
      "inner_hits": {} 
    }
  }

PUT test/_doc/1?refresh
{
  "title": "Test title",
  "comments": [
    {
      "author": "kimchy",
      "text": "comment text",
      "votes": []
    },
    {
      "author": "nik9000",
      "text": "words words words",
      "votes": [
        {"value": 1 , "voter": "kimchy"},
        {"value": -1, "voter": "other"}
      ]
    }
  ]
}

POST test/_search
{
  "query": {
    "nested": {
      "path": "comments.votes",
        "query": {
          "match": {
            "comments.votes.voter": "kimchy"
          }
        },
        "inner_hits" : {}
    }
  }
}

--Multi Level Nested
PUT /drivers
{
  "mappings": {
    "properties": {
      "driver": {
        "type": "nested",
        "properties": {
          "last_name": {
            "type": "text"
          },
          "vehicle": {
            "type": "nested",
            "properties": {
              "make": {
                "type": "text"
              },
              "model": {
                "type": "text"
              }
            }
          }
        }
      }
    }
  }
}
PUT /drivers/_doc/1
{
  "driver" : {
        "last_name" : "McQueen",
        "vehicle" : [
            {
                "make" : "Powell Motors",
                "model" : "Canyonero"
            },
            {
                "make" : "Miller-Meteor",
                "model" : "Ecto-1"
            }
        ]
    }
}

PUT /drivers/_doc/2?refresh
{
  "driver" : {
        "last_name" : "Hudson",
        "vehicle" : [
            {
                "make" : "Mifune",
                "model" : "Mach Five"
            },
            {
                "make" : "Miller-Meteor",
                "model" : "Ecto-1"
            }
        ]
    }
}
GET /drivers/_search
{
  "query": {
    "nested": {
      "path": "driver",
      "query": {
        "nested": {
          "path": "driver.vehicle",
          "query": {
            "bool": {
              "must": [
                { "match": { "driver.vehicle.make": "Powell Motors" } },
                { "match": { "driver.vehicle.model": "Canyonero" } }
              ]
            }
          }
        }
      }
    }
  }
}
--Parent/Child Nested

has_child and has_parent queries
A join field relationship can exist between documents within a single index. 
The has_child query returns parent documents whose child documents match the specified query, 
while the has_parent query returns child documents whose parent document matches the specified query.

Child olan iki veriyi queryden sonra has_child diye bağlayıp hangi childda aryırosak type diye belirtiyoruz.

has_child: Childla matchleşen querydeki parent verileri döndürür.
Returns parent documents whose joined child documents match a provided query.
You can create parent-child relationships between documents in the same index using a join field mapping.
has_parent: parentla matchleşen querydeki childe verileri döndürür.
Returns child documents whose joined parent document matches a provided query. 
You can create parent-child relationships between documents in the same index using a join field mapping.

PUT test/_doc/1?refresh
{
  "number": 1,
  "my_join_field": "my_parent"
}

PUT test/_doc/2?routing=1&refresh
{
  "number": 1,
  "my_join_field": {
    "name": "my_child",
    "parent": "1"
  }
}

POST test/_search
{
  "query": {
    "has_child": {
      "type": "my_child",
      "query": {
        "match": {
          "number": 1
        }
      },
      "inner_hits": {}    
    }
  }
}
"query": {
    "has_child": {
      "type": "child",
      "query": {
        "match_all": {}
      },
      "max_children": 10,
      "min_children": 2,
      "score_mode": "min"
    }
  }

(type ile child join işlemi gerçekleşir.)
(parent_type ile parent join işlemi gerçekleşir.)

"query": {
    "has_parent": {
      "parent_type": "parent",
      "query": {
        "term": {
          "tag": {
            "value": "Elasticsearch"
          }
        }
      }
    }
  }

-Field yapısı
Bu yapı tablolardan dönmesini istediğimiz kolonları dönmek için kullanılır.
PUT my-index-000001/_doc/1?refresh=true
{
  "group" : "fans",
  "user" : [
    {
      "first" : "John",
      "last" :  "Smith"
    },
    {
      "first" : "Alice",
      "last" :  "White"
    }
  ]
}

POST my-index-000001/_search
{
  "fields": ["*"],
  "_source": false
}
-----
POST my-index-000001/_search
{
  "fields": ["user.first"],
  "_source": false
}

 "fields": {
        "user": [{
            "first": ["John"]
          },
          {
            "first": ["Alice"]
          }
        ]
      }
	 
Sort İşlemleri(Sıralama)
order: Sıralama işlemi.
 - asc, desc
modlar
	- min
	- max
	- sum
	- avg
	- median
PUT /my-index-000001/_doc/1?refresh
{
   "product": "chocolate",
   "price": [20, 4]
}

POST /_search
{
   "query" : {
      "term" : { "product" : "chocolate" }
   },
   "sort" : [
      {"price" : {"order" : "asc", "mode" : "avg"}}
   ]
}
Nested objelerde sort işlemleri
POST /_search
{
   "query": {
      "nested": {
         "path": "parent",
         "query": {
            "bool": {
                "must": {"range": {"parent.age": {"gte": 21}}},
                "filter": {
                    "nested": {
                        "path": "parent.child",
                        "query": {"match": {"parent.child.name": "matt"}}
                    }
                }
            }
         }
      }
   },
   "sort" : [
      {
         "parent.child.age" : {
            "mode" :  "min",
            "order" : "asc",
            "nested": {
               "path": "parent",
               "filter": {
                  "range": {"parent.age": {"gte": 21}}
               },
               "nested": {
                  "path": "parent.child",
                  "filter": {
                     "match": {"parent.child.name": "matt"}
                  }
               }
            }
         }
      }
   ]
}
