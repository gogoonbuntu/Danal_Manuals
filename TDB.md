Danal 휴대폰결제 연동 가이드
=========================

# 사전 준비사항
### 방화벽 설정 : Outbound 방화벽 사용하는 CP일 경우
서버|IP|PORT
---|---|---
CP 인증서버| trans.teledit.com | 31000
결제거래서버| 211.170.89.1 ~ 211.170.89.15 <br/> 150.242.133.113 ~ 150.242.133.116 |5505
취소서버| trans.teledit.com | 13003 
웹 서버| ui.teledit.com | 443 

### PHP 용 연동모듈 (gcc 필요)
### JSP 용 연동모듈
### ASP/NET 용 연동모듈
<br>

# 연동 순서
1. 연동모듈 설치
  * 소켓통신&암호화 모듈 입니다. 해당 모듈에 파라미터를 담아 요청하고 응답을 받습니다.
2. CP 정보 입력 및 경로 수정 (Ready.*, function.*)
3. 결제완료 후 (CPCGI.*) CP 작업 시행 (서비스 제공, TID 저장, DB 관련 작업 등)
<br>

# 서비스 연동 플로우
총 5단계로 진행되며, B와 C 단계는 다날 웹서버 (다날 표준 결제창)을 사용하여 연동하게 된다.
[diagram]

<br>

# 각 페이지 설명
|파일|설명|
|---|---|
|./inc/function.*|공통 Function 정의|
|Ready.*|CP 및 상품정보 전송 페이지|
|CPCGI.*|결제 최종 요청 페이지|
|Error.*|에러 페이지|
|Success.*|결제 성공화면(UI) 페이지|
|BillCancel.*|결제 취소 요청 페이지|

## function 페이지
1. 역할 : 휴대폰결제 서비스에 필요한 각종 Function 정의
2. 수정사항
> ID : 다날에서 제공해드린 CPID   
> PWD : 다날에서 제공해드린 PWD   
> AMOUNT : 결제 금액

## Ready 페이지
1. 역할 : CP인증 (A단계)
2. 설명 : CP정보(ID, PWD) 및 상품정보(ItemAmt, ItemName, ItemCode 등)을 서버로 전송 후 결제창을 호출함.
  1. "다날 인증서버"와 통신하여 가맹점 인증을 받는다.
  2. 인증 성공시, "다날 웹서버"로 FORM 태그를 HTTP POST방식으로 전송하여 결제창을 호출한다.
### 인증 단계 파라미터
#### INPUT
|Field|필수여부|Max(Byte)|비고(예제)|
|---|---|---|---|
|CP 정보|||
|ID|필수|10|CP ID|
|PWD|필수|10|CP PWD|
|SUBCP|선택사항| |Sub CPID (사용하는 경우만)|
|사용자 정보||||
|USERID|선택사항|60|고객 ID|
|상품 정보|||
|ItemCount|필수|-|고정값

#### OUTPUT
Field|필수여부|Max(Byte)|비고(예제)
---|---|---|---
Result|필수|4|결과코드
ErrMsg|필수|256|결과메시지
ServerInfo|필수|128|거래인증 key(거래세션)

### 결제창 호출 단계 파라미터
#### INPUT (CP WEB SERVER → DANAL WEB SERVER)
Field|필수여부|Max(Byte)|비고(예제)
---|---|---|---
ServerInfo | 필수 | 다날 인증서버에서 받은 거래 키(128 byte)
CPName | 필수 | 결제화면에 표시되는 **가맹점 이름**
ItemName | 필수 | 결제화면에 표시되는 **상품 이름**
ItemAmt | 필수 | 결제화면에 표시되는 **결제금액 **
TargetURL | 필수 | 최종 결제 요청할 CP의 CPCGI FULL URL <br> (https://cp.site.co.kr/xxx/xxx/CPCGI.*)
BackURL | 선택사항 | 에러 발생 및 취소 시 이동할 페이지의 FULL URL <br> (https://cp.site.co.kr/xxx/xxx/BackURL.*)
IsUseCI | 선택사항 | 가맹점 CI 사용여부
CIURL | 선택사항 | 가맹점 CI 이미지 페이지 FULL URL<br>(https://cp.site.co.kr/xxx/xxx/cpci.jpg)<br><br>CI 이미지 사이즈 = 119x47
IsPreOtbill | 선택사항 | 자동결제, Authkey 수신 여부(Y/N) <br> (결제화면 내 제공기간 항목에 표기)
IsSubscript | 선택사항 | 월 정액 가입 여부(Y/N) <br> (결제화면 내 제공기간 항목에 표기)
IsCharSet | 선택사항 | CP 측 웹서버 인코딩 방식
BgColor | 선택사항 | 결제화면 Color 설정시 사용 00:블루, 01:남색, 02:노랑, 03:민트, 04:보라, 05:분홍, 06:빨강, 07:주황, 08:청록, 09:초록, 10:회색
Email | 선택사항 | 사용자 E-mail주소 = 결제화면에 표기
ByPassValue | 선택사항 | CPCGI 페이지로 전달될 데이터
UseAuthKey | 선택사항 | 사용자 인증 키 전달 여부 <br> 'Y' 전달 시 사용자 인증 완료 시 전달
#### OUTPUT (DANAL WEB SERVER → CP WEB SERVER)
Field|필수여부|Max(Byte)|비고(예제)
---|---|---|---

