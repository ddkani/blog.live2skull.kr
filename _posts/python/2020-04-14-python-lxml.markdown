---
layout: post
title:  "파이썬에서 XPath를 이용한 HTML문서 파싱"
date:   2020-04-14 21:00:00 +0900
categories: python lxml html
---

파이썬에서 `lxml` 패키지로 `XPath`를 이용해 HTML문서를 파싱하는 법을 다루어 봅니다.

`XPath`를 이용하면 검색된 엘리멘트를 코드로 재검색하는 과정을 `XPath` 문법에 작성하여 보다 간결하게 파싱 과정을 구현 할 수 있습니다.

HTML문서의 파싱은 `BeautifulSoup` 패키지를 이용하는 방법이 많이 소개되어 있습니다. 간결하고 사용하지 편리하지만, `XPath`를 자원하지 않으므로, 이러한 경우 대응책으로 `lxml`을 사용할 수 있습니다.


### lxml 패키지 설치
다음 명령어로 `lxml`을 설치합니다.
```
pip3 install lxml
```

### HTML문서 로드
로컬에 저장된 파일이나 인터넷의 URL에서 문서를 로드합니다.
```
import lxml.html
from lxml.html import HtmlElement

html_element = lxml.html.parse(
	DOCUMENT_PATH_OR_URL
) # type: HtmlElement
```

이미 로드된 str() 문자열로 로드할 수 도 있습니다.
```
import lxml.html
from lxml.html import HtmlElement

html_element = lxml.html.document_fromstring(
	LOADED_DOCUMENT_STR
) # type: HtmlElement
```

----

## XPath로 검색하기

#### 주요 문법 (expression)

`XPath`는 다양한 종류의 연산자로 구성되어 있으며, 이를 조합하여 원하는 객체를 선택합니다.

**경로 연산자**

|문법|설명|
|------|-------|
|/|루트 노드(문서의 시작), 재귀적으로 탐색하지 않음.
|//|지정한 노드에서 재귀적으로 탐색.
|.|현재 노드를 선택
|..|현재 노드의 부모 노드를 선택
|@|속성 노드를 선택 (HTML에서 attribute지정)

**방향 연산자**

엘리멘트를 검색하는 방향을 지정합니다.

|문법|설명|
|------|-------|
|self|현재 노드
|attribute|현재 노드의 속성 노드
|namespace|현재 노드의 네임스페이스 노드
|child|현재 노드의 자식 노드
|descendant|현재 노드의 자손 노드 (자식 -> 자손)
|descendant-or-self|현재 노드와 자손 노드
|following|현재 노드의 종료 이후 등장하는 노드
|following-sibling|현재 노드 이후 등장하는 형제 노드
|parent|현재 노드의 부모 노드
|ancestor|현재 노드의 조상 노드
|ancestor-or-self|현재 노드와 조상 노드
|preceding|현재 노드 이전의 모든 노드(조상,속성, 네임스페이스 노드)
|preceding-sibling|현재 노드 이전의 형제 노드

위 연산자를 경로 표현식(path expression)으로 조합하여 검색합니다.
```
방향연산자::경로연산자[필터표현식]
```

----

#### 예제

다음 예제를 통해 태그 이름과 경로, 속성값을 이용해 원하는 엘레멘트를 검색하는 기본적인 방법을 확인할 수 있습니다.

\#1. 문서의 전체에서 div태그 중 id가 'articleHead'인 태그의 자식 (자손이 아닌) span태그인 엘리멘트 중 두번째를 선택합니다.
```
//div[@id='articleHead']/span[2]
```

\#2. 문서의 전체에서 div태그 중 id가 'articleBodyContents' 이고 class가 '\_article_body_contents'인 엘레멘트를 찾습니다.
```
//div[@id='articleBodyContents' and @class='_article_body_contents']
```

\#3. 문서의 전체에서 disabled속성을 가진 모든 엘레멘트를 찾습니다.
```
//*[@disabled]
```

\#4. 문서의 전체에서 이미지 태그 중, 도메인이 `blog.live2skull.kr` 인 엘레멘트를 찾습니다.
```
//img[contains(@href, 'blog.live2skull.kr')]
```
----

#### 실행 결과

주요 문법을 상황에 따라 적절히 조합하여, 원하는 엘리멘트를 추출합니다. `XPath`에 따른 리턴값은 다음과 같습니다.

\#1. 객체를 선택하는 경우 `list` 입니다. 특정 인덱스를 지정하여 **하나의 객체만 선택** 하여도, 반환 형식은 동일합니다.

```
>>> elem.xpath('//div')
[<Element div at 0x108f855f0>, <Element div at 0x108f85110>, ... ]
>>> elem.xpath('//div[1]')
[<Element div at 0x108f855f0>]
```
\#2. `string()` 과 같은 문자열 출력은 객체 하나만을 반환합니다.
```
>>> elem.xpath('string()')
... \';\nvar _MAIN_NEWS_MENU_ID=\'shm\';\nvar _MAIN_NEWS_SEC ...
>>> type(elem.xpath('string()'))
<class 'lxml.etree._ElementUnicodeResult'> # str()와 동일히 사용 가능
```

## 엘리멘트 값 추출

원하는 엘리멘트를 찾았다면, 필요한 값을 추출합니다.

#### 태그 이름

```
>>> img_elem.tag
'img'
```

#### 속성 (attribute)

```
>>> img_elem.attrib
{'src' : 'https://blog.live2skull.kr/...', ... } # dict 와 동일히 사용 가능
>>> img_elem.attrib['src']
'https://blog.live2skull.kr/...'
```


#### 엘리멘트 문자열 구하기
엘리멘트의 자식 노드까지의 텍스트 구하기
```
message = element.text_content()
```


#### 전체 문자열 데이터

⚠️ 내부 구현을 확인하지 못했습니다. `BeautifulSoup` 에서의 `.string` 내부 문자열 추출 메서드와 다른 결과가 발생할 수 있습니다.

⚠️ 태그에 삽입된 텍스트가 '\n'이 아닌 `BR` 태그로 구분되었다면, 줄바꿈이 되지 않습니다.

⚠️ `.text` 메서드가 상황에 따라 의도한 대로 동작하지 않을 수 있습니다.

노드와 노드의 전체 자식 노드 중 comment 엘리멘트와 일부를 제외하고, `<div>general message</div>` 와 같이 태그 내부에 삽입된 문자열을 반환합니다.

```
>>> html_element = lxml.html.document_fromstring(
	'<div>general message</div>'
)
>>> html_element.text_content()
'general message'
```

#### 부모 / 자식 노드, 이전 / 다음 노드 접근
`getparent()` 부모 노드 접근
```
>>> element.getparent()
<Element div at 0x108f85a70>
```

`getchildren()` 자식 노드 접근. 반환 형식은 `list` 입니다. for 문으로 바로 사용할 수 도 있습니다.
```
>>> element.getchildren()
[<Element div at 0x108f85fb0>, <Element div at 0x108f85cb0>]

# 객체에 iteration을 지원으므로 for문에서 바로 사용 가능합니다.
>>> for elem in element: print(elem)
<Element div at 0x108f85fb0>
...
```
마찬가지로 `getprevious()` `getnext()` 로 이전, 다음 노드에 접근 가능하며, 접근 가능한 노드가 없다면 `None`을 반환합니다.

----

## 실수하기 쉬운 코드

💡 `XPath`의 인덱스 시작은 **1** 입니다.

💡 문서를 로드시 반환되는 `_ElementTree` 의 루트는 `body` 엘리멘트가 아닙니다. `_ElementTree->head,body-> ...` 입니다.

💡 검색된 노드에서 다시 검색하고자 할 경우 `XPath`에 상대경로를 지정해주어야 합니다. 그렇지 않으면 검색된 노드 내에서 검색하지 않고, 전체 문서에서 검색하게 됩니다.  
상대 경로로 표시하기: **'.//USER_DEFINED_PATH'** - 경로의 시작에 '.' 을 붙입니다.
```
html_element = lxml.html.parse(DOCUMENT_PATH) # root element
article_element = html_element.xpath('//div[@id="articleInfo"]')[0]

## article_element 내부에서 검색하려고 하므로, 상대 경로를 지정합니다.
title_element = article_element.xpath('.//span[@id="title"]')[0]

## 만약 상대 경로를 지정하지 않았다면, 문서 전체에서 찾게 됩니다.
title_element = article_element.xpath('//span[@id="title"]')[0]
```


💡 불필요한 엘리멘트를 삭제할 때는 반드시 대상 엘리멘트의 부모 엘리멘트에서 삭제하여야 합니다. 또한, 파싱된 최상위 루트 엘리멘트에서는 삭제가 불가능합니다. (`_ElementTree` 에서 삭제 불가 / `HtmlElement` 에서 삭제 가능)
```
>>> a.remove(b) # a 노드에서 b 노드를 삭제하는 예제
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "src/lxml/etree.pyx", line 943, in lxml.etree._Element.remove
ValueError: Element is not a child of this node.
# a가 b의 바로 상위 노드가 아니므로, 삭제할 수 없음.
```
