---
title: Topic models
weight: 50
draft: false
---

Topics models are unsupervised document classification techniques. By modeling distributions of topics over words and words over documents, topic models identify the most discriminatory groups of documents automatically. 


```r
require(quanteda)
require(quanteda.corpora)
require(seededlda)
require(lubridate)
```


```r
corp_news <- download("data_corpus_guardian")
```



We only select news articles published in 2016 using `corpus_subset()` and the `year` function from the **lubridate** package. 


```r
corp_news_2016 <- corpus_subset(corp_news, year(date) == 2016)
ndoc(corp_news_2016)
```

```
## [1] 1959
```

Further, after removal of function words and punctuation in `dfm()`, we keep only the top 5% of the most frequent features (`min_termfreq = 0.8`) that appear in less than 10% of all documents (`max_docfreq = 0.1`) using `dfm_trim()` to focus on common but distinguishing features.


```r
toks_news <- tokens(corp_news_2016, remove_punct = TRUE, remove_numbers = TRUE, remove_symbol = TRUE)
toks_news <- tokens_remove(toks_news, pattern = c(stopwords("en"), "*-time", "updated-*", "gmt", "bst"))
dfmat_news <- dfm(toks_news) %>% 
              dfm_trim(min_termfreq = 0.8, termfreq_type = "quantile",
                       max_docfreq = 0.1, docfreq_type = "prop")
```

**quanteda** does not implement topic models, but you can fit LDA and seeded-LDA with the **seededlda** package.

### LDA

`k = 10` specifies the number of topics to be discovered. This is an important parameter and you should try a variety of values and validate the outputs of your topic models thoroughly.


```r
tmod_lda <- textmodel_lda(dfmat_news, k = 10)
```

You can extract the most important terms for each topic from the model using `terms()`.


```r
terms(tmod_lda, 10)
```

```
##       topic1      topic2     topic3    topic4    topic5      topic6    
##  [1,] "oil"       "officers" "climate" "clinton" "doctors"   "syria"   
##  [2,] "markets"   "prison"   "water"   "sanders" "violence"  "isis"    
##  [3,] "sales"     "victims"  "energy"  "cruz"    "nhs"       "military"
##  [4,] "prices"    "criminal" "food"    "hillary" "education" "un"      
##  [5,] "banks"     "dead"     "gas"     "obama"   "medical"   "islamic" 
##  [6,] "investors" "charges"  "project" "trump's" "hospital"  "syrian"  
##  [7,] "rates"     "incident" "china"   "bernie"  "drug"      "muslim"  
##  [8,] "sector"    "officer"  "air"     "ted"     "schools"   "forces"  
##  [9,] "shares"    "arrested" "chinese" "rubio"   "child"     "russian" 
## [10,] "trading"   "black"    "land"    "senator" "study"     "peace"   
##       topic7     topic8       topic9       topic10       
##  [1,] "game"     "corbyn"     "australia"  "refugees"    
##  [2,] "apple"    "johnson"    "australian" "brussels"    
##  [3,] "facebook" "leadership" "labor"      "talks"       
##  [4,] "google"   "shadow"     "turnbull"   "french"      
##  [5,] "users"    "boris"      "senate"     "summit"      
##  [6,] "games"    "jeremy"     "budget"     "migrants"    
##  [7,] "music"    "tory"       "coalition"  "greece"      
##  [8,] "play"     "khan"       "funding"    "benefits"    
##  [9,] "iphone"   "scotland"   "malcolm"    "refugee"     
## [10,] "video"    "commons"    "laws"       "negotiations"
```

You can then obtain the most likely topics using `topics()` and save them as a document-level variable.


```r
head(topics(tmod_lda), 20)
```

```
## text136751 text136585 text139163 text169133 text153451 text163885 text157885 
##     topic7     topic8     topic7     topic2     topic2     topic3     topic7 
## text173244 text137394 text169408 text184646 text127410 text134923 text169695 
##     topic3     topic9     topic7     topic8     topic2     topic8     topic1 
## text147917 text157535 text177078 text174393 text181782 text143323 
##     topic7     topic3     topic3     topic4     topic5     topic8 
## 10 Levels: topic1 topic2 topic3 topic4 topic5 topic6 topic7 topic8 ... topic10
```

```r
# assign topic as a new document-level variable
dfmat_news$topic <- topics(tmod_lda)

# cross-table of the topic frequency
table(dfmat_news$topic)
```

```
## 
##  topic1  topic2  topic3  topic4  topic5  topic6  topic7  topic8  topic9 topic10 
##     262     253     203     203     209     180     204     207     161      70
```

### Seeded LDA

In the seeded LDA, you can predefine topics in LDA using a dictionary of "seed" words.


```r
# load dictionary containing seed words
dict_topic <- dictionary(file = "../dictionary/topics.yml")
print(dict_topic)
```

```
## Dictionary object with 5 key entries.
## - [economy]:
##   - market*, money, bank*, stock*, bond*, industry, company, shop*
## - [politics]:
##   - lawmaker*, politician*, election*, voter*
## - [society]:
##   - police, prison*, school*, hospital*
## - [diplomacy]:
##   - ambassador*, diplomat*, embassy, treaty
## - [military]:
##   - military, soldier*, terrorist*, marine, navy, army
```

The number of topics is determined by the number of keys in the dictionary. Next, we fit the seeded LDA model using `textmodel_seededlda()` and specify the dictionary with our relevant keywords.


```r
tmod_slda <- textmodel_seededlda(dfmat_news, dictionary = dict_topic)
```

Some of the topic words are seed words, but the seeded LDA identifies many other related words.


```r
terms(tmod_slda, 20)
```

```
##       economy       politics      society       diplomacy    military    
##  [1,] "markets"     "politicians" "hospital"    "treaty"     "military"  
##  [2,] "banks"       "elections"   "schools"     "ambassador" "terrorist" 
##  [3,] "stock"       "politician"  "prison"      "diplomatic" "army"      
##  [4,] "banking"     "voter"       "hospitals"   "diplomats"  "soldiers"  
##  [5,] "shop"        "lawmakers"   "prisons"     "diplomat"   "terrorists"
##  [6,] "shopping"    "clinton"     "prisoners"   "embassy"    "navy"      
##  [7,] "bank's"      "sanders"     "hospitality" "labor"      "marine"    
##  [8,] "marketing"   "cruz"        "prisoner"    "corbyn"     "soldier"   
##  [9,] "shops"       "obama"       "violence"    "johnson"    "refugees"  
## [10,] "bankers"     "hillary"     "officers"    "turnbull"   "syria"     
## [11,] "stocks"      "trump's"     "cases"       "budget"     "isis"      
## [12,] "shoppers"    "bernie"      "parents"     "cabinet"    "un"        
## [13,] "bond"        "senator"     "sexual"      "australian" "syrian"    
## [14,] "bonds"       "ted"         "abuse"       "benefits"   "turkey"    
## [15,] "bankruptcy"  "rubio"       "drug"        "talks"      "islamic"   
## [16,] "bankrupt"    "gun"         "facebook"    "brussels"   "aid"       
## [17,] "banker"      "primary"     "victims"     "shadow"     "forces"    
## [18,] "stockport"   "race"        "child"       "coalition"  "border"    
## [19,] "marketplace" "candidates"  "officer"     "leadership" "french"    
## [20,] "oil"         "kasich"      "mental"      "australia"  "refugee"
```

`topics()` on returns dictionary keys as the most likely topics of documents.


```r
head(topics(tmod_slda), 20)
```

```
## text136751 text136585 text139163 text169133 text153451 text163885 text157885 
##    economy  diplomacy    economy    society    society    economy   politics 
## text173244 text137394 text169408 text184646 text127410 text134923 text169695 
##   military  diplomacy    society  diplomacy    society  diplomacy    economy 
## text147917 text157535 text177078 text174393 text181782 text143323 
##   politics    economy    economy    society    society  diplomacy 
## Levels: economy politics society diplomacy military
```

```r
# assign topics from seeded LDA as a document-level variable to the dfm
dfmat_news$topic2 <- topics(tmod_slda)

# cross-table of the topic frequency
table(dfmat_news$topic2)
```

```
## 
##   economy  politics   society diplomacy  military 
##       497       218       564       353       320
```

{{% notice ref %}}

- Blei, David M., Andrew Y. Ng, and Michael I. Jordan. 2003. "[Latent Dirichlet Allocation](https://www.jmlr.org/papers/volume3/blei03a/blei03a.pdf)." _The Journal of Machine Learning Research_ 3(1): 993-1022.  
- Lu, B., Ott, M., Cardie, C., & Tsou, B. K. 2011. "[Multi-aspect Sentiment Analysis with Topic Models](https://www.cs.cornell.edu/home/cardie/papers/masa-sentire-2011.pdf)". _Proceeding of the 2011 IEEE 11th International Conference on Data Mining Workshops_, 81–88.

{{% /notice %}}

