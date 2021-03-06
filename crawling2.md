**Step 5-1: NAVER 의약품사전 웹 크롤링**
===
---
### 1. 개발 환경 설정
크롤링에 필요한 모듈을 설치해주세요.
~~~
$npm init
$npm install request
$npm install cheerio
~~~

### 2. 웹 크롤러 작성

2.1. 크롤링해올 페이지 단위수와 함께, 인코딩과 uri 옵션을 설정해주세요.
~~~
for(var page_num=1; page_num<=10; page_num++){

var options = {
  encoding: "UTF-8",
  uri: 'http://terms.naver.com/medicineSearch.nhn?page=' + page_num
}
~~~

2.2. 목차 안에 있는 글들의 URL을 추출하는 작업을 합니다.

![pic2_2](https://user-images.githubusercontent.com/25143478/36528743-81c1f8a0-17f9-11e8-991b-80452b0cc698.JPG)<br>
content_list 태그를 받아와서 li가 하나의 글을 의미합니다.
info_area 이름의 클래스안에 있는 href에서 url을 받아옵니다.  
~~~
$('.content_list li').each(function() {
    var $table_url_temp = $(this).find(".info_area").find('a').attr('href');
    var $table_url = 'http://terms.naver.com/' + $table_url_temp;
~~~

2.3. 약 이름, 효능효과, 용법용량, 주의사항, 이미지URL을 클래스 이름과 ID로 구분해 가져옵니다.

![pic2_3](https://user-images.githubusercontent.com/25143478/36528750-83030ef2-17f9-11e8-94ba-2d165b7b5610.JPG)<br>

예를 들어 효능효과의 경우, 효능효과 제목 다음의 태그가 실질적인 내용을 담고 있기 때문에, next()로 다음 태그를 접근해줍니다.
~~~
var options2 = {
    encoding: "UTF-8",
    uri: $table_url
}

request(options2, function(error, response, html){
  if (error) {throw error};

  var $ = cheerio.load(html);
  var $drug_name = $(".headword").text();
  var $drug_efficacy = $("#TABLE_OF_CONTENT4").next().text();
  var $howtouse = $("#TABLE_OF_CONTENT5").next().text();
  var $precaution = $("#TABLE_OF_CONTENT6").next().text();
  var $url = $table_url;
  var $img_url = $('.img_box').find('img').attr('data-src');
  var $synonyms = Hangultest.hanguler($drug_name);
~~~

2.4. 콘솔에 결과를 가져옵니다.

![pic2_4](https://user-images.githubusercontent.com/25143478/36528788-a0d3654e-17f9-11e8-92a9-9e6a2b9fcd2b.JPG)<br>

추후 약 정보 DB 테이블의 id자리를 남기기 위해 임의의 열 구분자 '!'를 추가합니다.

id자리 공백, 약 이름, Tolerance, 효능효과, 용법, 주의사항, 글URL, 이미지URL 순서로 한 행씩 묶어 추출합니다.



~~~
console.log("!"+$drug_name+"!"+$synonyms+"!"+$drug_efficacy+"!"+$howtouse+"!"+$precaution+"!"+$url+"!"+$img_url);
~~~

### 3. 데이터 저장
10페이지씩 나눠 text1~text10까지 텍스트 파일로 저장합니다.
그 다음 10개의 텍스트파일을 하나로 합칩니다.
~~~
$node medicine_cr.js > text1.txt
.
.
.
$node medicine_cr.js > text10.txt

$type text*.txt > text_total.txt
~~~

### 4. 데이터 정제
','는 나중에 csv파일로 변환될때 열 구분자가 됩니다.
따라서 ','문자는 '¸'으로 치환하고
열을 구분하기 위해 임의로 설정한 '!'문자는 ',' 으로 치환해줍니다.

### 5. csv 파일로 변환
![pic5](https://user-images.githubusercontent.com/25143478/36528799-ad3c1308-17f9-11e8-8dfe-dbec07cbdb42.JPG)<br>

.csv 확장자로 저장하되 반드시 UTF-8 형식으로 저장해주어야 합니다.


###Step 3 넘어가기 : [Step 3 - FAQ 페이지크롤링](https://github.com/minD3D/MyDoctor-Guide/blob/master/crawling3.md)
