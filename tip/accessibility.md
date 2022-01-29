# accessibility(접근성)

## accessibility(접근성) 이란
[**accessibility**] 유저의 내/외부적인 환경과 상관없이 문서가 전달하고자 하는 정보를 정확하게 전달하고자 하는것을 접근성을 높인다 라고 말한다. 갈수록 브라우저가 접근성을 지원하는 영역이 넒어지면서 기존엔 대체텍스트로 사용하던것들을 적절한 값을 변경해줌으로써 코드의 간결성은 물론이고 다양한 언어환경에서도 웹의 접근성을 높일수 있게 되었다. 접근성을 높이는 방법은 여러가지 방법이 있겠지만, HTML, 때론는 JS를 통해 접근성을 높이는 방법을 정리하고자 한다. 접근성에 주로 사용하는 role 과 각종 attribute 값을 실제 코드에 녹여서 정리하고 한다.

### 개념
#### role 
role 을 알고 시작 하면 좋다. role 이란 단순 div 나 span 으로 되어있는 아이들이 어떤 의미있는 영역을 나타내고자 할때 원래 section, article 이나 button 등등 적절한 html5 태그를 사용한다. 하지만 적절한 태그를 사용하지 못해 영역자체의 의미전달이 어려울때 role 의 attribute 를 사용한다.

##### 주로사용하는 role
	- search, tab, 
	- button, radio,
	- main, footer, presentation 
