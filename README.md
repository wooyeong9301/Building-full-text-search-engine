# Building-full-text-search-engine

Building a full-text search engine in 150 lines of Python code   
Ref : https://bart.degoe.de/building-a-full-text-search-engine-150-lines-of-code/

---

이 프로젝트에서 full-text 검색 엔진의 기초적인 구성 요소들을 알아보고, 수백만 개 문서 사이에서 검색하고 밀리초만에 관계도에 따라 랭크를 매기는 검색 엔진을 만들어 볼 것이다.   

Requirements   

- lxml==4.5.2

- PyStemmer==2.0.1

- requests==2.24.0

## Data

`run.py`를 실행시키면 데이터를 다운로드하고, 예시 쿼리를 수행한다.   

검색 엔진을 만들기 전에, 먼저 검색을 위한 구조화되지 않은 full-text 데이터가 필요하다. 영어 위키피디아에서 약 627만 개 기사의 요약본들을 검색할 것이다. `download.py`를 통해 다운로드할 수 도 있고, 직접 받을 수도 있다.

#### Data Preparation

데이터 파일은 모든 요약본들을 포함하고 있는 하나의 큰 XML 파일이다. 하나의 요약본은 `<doc>` 요소로 포함되어있다. 데이터에 편하게 접근하기 위해 문서들을 [Python dataclass](https://realpython.com/python-data-classes/)로 표현할 것이다. 그리고 제목과 내용을 병합한 property를 추가할 것이. `title_documents.py`   

그 후, `Abstract` 오브젝트의 인스턴스들을 만들기 위해  XML로부터 요약본들의 데이터를 추출한다. 먼저 전체 파일을 메모리에 로드하지 않고 gzipped XML을 스트림한다. 각 문서에는 로딩 순서대로 'ID'를 부여한다. `load.py`

## Indexing

데이터는 "Inverted Index" 혹은 "Postings List"로 알려진 데이터 구조로 저장한다. 각 단어들을, 단어가 포함되는 문서의 ID들과 맵핑하는 딕셔너리를 만들면 된다.    

인덱스를 만들기 전에, 텍스트를 단어들로 `tokenize`하고, 필요하다면 각 token에 `filters`를 적용하여 쿼리와 텍스트의 매칭에 문제가 없도록 해주는 과정이 필요하다.

#### Analysis

공백을 기준으로 문서를 나누는 간단한 토큰화를 진행한다. 그리고 각 토큰에 대해 소문자로 만들고, 구두점을 없애고, 영어에서 가장 통상적인 단어 25개와 'wikipedia'를 불용어로 직접 만들어 없애고, 어간 추출(stemming)을 진행한다. 주의할 점은, 어간 추출이 적용되지 않은 불용어 리스트를 사용하기 때문에, 어간 추출 필터를 적용하기 전에 불용어를 없애야 하는 것이다.   

---

*참고자료에선 각 필터를 함수로 나누었는데 굳이 그렇게 하지 않고 하나의 함수로 처리하면??*

*nltk 혹은 spacy의 stopwords를 사용하는 경우도 만들어보기*

---

#### Indexing the corpus

`index`와 `documents`를 저장할 `Index` 클래스를 만든다. `documents` 딕셔너리는 ID를 기준으로 데이터 클래스를 저장하고, `index`의 key는 토큰이고, value는 토큰이 등장하는 문서들의 ID이다.   

## Searching
