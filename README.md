야옹아 멍멍해봐 : 공공데이터를 이용한 유기견 입양 서비스
===========================================
* 유기동물 입양를 할 수 있는 웹 사이트 입니다.    
* 서울시에서 제공하는 공공데이터로 유기동물의 정보와 현황을 알 수 있고 하루 1회 업데이트 됩니다.    
* 입양 전 관심리스트를 만들어 관리 할 수 있습니다.    
* 사람들과의 커뮤티니 활동을 할  수 있습니다.

### 시연 영상
https://github.com/minminjee/202401_animal_java_project/assets/157664207/22916300-a821-4e43-aa4f-b3752512e3a0

  
### 개발 구상도
<img src ="https://github.com/minminjee/animal_java_project_24.01/assets/157664207/7871258a-3466-498a-a129-0d96c4dd9ccb">


### 개발 구상도_상세
<img src ="https://github.com/minminjee/animal_java_project_24.01/assets/157664207/c9d0fd3a-ae04-4438-b7b3-65a5fd6860e7">


### 개발동기
<img src ="https://github.com/minminjee/animal_java_project_24.01/assets/157664207/9e84e493-d736-47e8-8b36-806b1a4897ee">


### ERD
<img src="https://github.com/minminjee/animal_java_project_24.01/assets/157664207/85f162d3-91fb-4458-b4e5-10beb604f0d6">

### 기술 스택


#### 1. 공공 데이터 api 파싱    
api를 사용하기 위해서는 내가 필요한 형식로 정보를 받고 파싱해서 사용한다.    
제공하는 API 형식 중에 내가 구현하려는 데이터에는 key :value가 적합하여서  JSON형태로 받았다.     
넘겨주는 데이터는 형식은 json이지만 String으로 되어있어서 데이터에 접근하기 위해서 파싱을 하였다.     
파싱된 데이터로 사진과 동물에 관련된 정보를 출력하기 용이하였고 for문을 사용하여 모든 정보에 대한 순차적인 출력도 가능하였다.      
사용하려는 api 데이터가 두개로 나눠져있다. 사진리스트와 정보리스트가 각각의 key value로 만들어져있다. (사진은 여러개)      
이걸 2차 배열로 만들었었는데 출력할때마다 너무 비효율적이다. 한번에 출력이 가능하도록 구현해보자 기능면에서는 동일 할 것이다. 

**문제**     
사용하려는 두 api를 2차배열로 만들었지만 서버로 데이터 전송하기 복잡
api를 매칭하며 새로운 key에 이미지 사진 배열 저장함


**수정 후 로직 비교**    
다시 배열에서 이미지를 가져오기 때문에 속소가 느리지만
서버로 json 형태로 전송이 간편

```
$(document).ready(function(){
		//이미지 API 가져오기
		$.ajax({
			async : true,
			type : 'GET',
			url : 'animalPhotoAPI',
			contentType : "application/json; charset=UTF-8",
			success : function(photoData){
				console.log("photo success");
				
				//정보API 가져오기
				$.ajax({
					async : true,
	                type : 'GET',
	                url : 'animalInfoAPI',
	                contentType : "application/json, charset=UTF-8",
	                success : function(infoData){
	                    console.log("info success");
	                    
									var parsedPhotoData = JSON.parse(photoData); //이미지 API json parsing
	                var parsedInfoData = JSON.parse(infoData); //정보 API json parsing
	
	                var photoRows = parsedPhotoData.TbAdpWaitAnimalPhotoView.row; //이미지 API key find
	                var infoRows = parsedInfoData.TbAdpWaitAnimalView.row; //정보 API key find
	
	                 //이미지 API와 정보 API 동물번호를 기준으로 매칭하여 
	                 //정보 API에 이미지 정보들을 저장하는 key 생성

	                    for(var i=0; i<infoRows.length; i++){
	                        var animalInfoNo = infoRows[i].ANIMAL_NO;
	                        let imageArr = [];
	                        for(var j=0; j<photoRows.length; j++){
	                            var animalPhotoNo = photoRows[j].ANIMAL_NO;
	                            if(animalInfoNo == animalPhotoNo){
	                                imageArr.push(photoRows[j].PHOTO_URL);
	                                //break;
	                            }
	                        }
	                        infoRows[i].PHOTO_LIST = imageArr;
	                    }
	                    
	                    var jsonFile = JSON.stringify(infoRows);
	                   
	                    //브라우저 로컬스토리지 내 api 모든 정보 저장
	                    localStorage.setItem('jsonFileAll',jsonFile);
```
#### 2. session 값 활용하기   
클라이언트의 정보를 저장하기 위해서는 

1. 매번 DB를 다녀오는 방법 > 더 많은 데이터를 필요로 할때
    
2. 클라이언트에 저장하는 방법 _ 쿠키값 > 중요하지 않은 정보들만 기록
    
3. 서버에 저장하는 방법 _ session값 > 로그인 정보 중요한 정보
    
나는 로그인 된 정보를 유지되고 있는 것처럼 구현하고자 sesion값 지정으로 구현하였다.   
로그인 정보 및 비회원과 회원의 유효성으로 접근 제한 기능들을 필요했다.    
view 부분에서는 변수로 접근이 가능하다.     
session은 모든 페이지에서 사용할 수 있는 멤버 변수와도 같다.     
그래서 선언(변수값 변경)을 여러 곳에서 할 시 복잡해지거나 값이 변경될 수 있으므로 선언에 신경 쓰도록하자.     
이번 프로젝트에서는 1곳에서 선언하고 1곳에서 재정의 하였다.     

```
	if(session.getAttribute("login")!=null) {
			session.removeAttribute("login");
		}
		
		MemberVO mvo = mservice.login(memberVo);
		
		if(mvo!=null) { // 로그인 성공
			session.setAttribute("login", mvo);

```

```
<c:if test="${login != null }">
                  <a href="mypage"> <span class="nickname">${login.member_nickname }님 어서오세요.</span></a>
                    <button class="member-logoutbtn"><a href="logout">
                    <span class="btn1">로그아웃</span><br>
                    </a></button>
             </c:if>
```
