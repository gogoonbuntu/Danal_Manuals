Danal 휴대폰결제 연동 가이드
=========================

# 사전 준비사항
### * 방화벽 설정 : Outbound 방화벽 사용하는 CP일 경우
서버|IP|PORT
---|---|---
CP 인증서버| trans.teledit.com | 31000
결제거래서버| 211.170.89.1 ~ 211.170.89.15 <br/> 150.242.133.113 ~ 150.242.133.116 |5505
취소서버| trans.teledit.com | 13003 
웹 서버| ui.teledit.com | 443 

### * 연동모듈 (PHP/JSP/JS/PYTHON/ASP/NET)
* PHP, JS, PYTHON 은 gcc 가 필요함
---


# 연동 순서
1. 연동모듈 설치
  * 소켓통신&암호화 모듈 입니다. 해당 모듈에 파라미터를 담아 요청하고 응답을 받습니다.
2. CP 정보 입력 및 경로 수정 (Ready.\*, function.\*)
3. 결제완료 후 (CPCGI.*) CP 작업 시행 (서비스 제공, TID 저장, DB 관련 작업 등)
<br>

# 서비스 연동 플로우
총 5단계로 진행되며, B와 C 단계는 다날 웹서버 (다날 표준 결제창)을 사용하여 연동하게 된다.
<br><br>
![image](https://user-images.githubusercontent.com/34636395/194971461-ad8eb0b6-537d-49ed-b9e9-a554ddb40b75.png)


<br>

---

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
* 역할 : 휴대폰결제 서비스에 필요한 각종 Function 정의

## I. Ready 페이지 - 가맹점 인증
* 역할 : CP인증 (A단계)
* 설명 : CP정보(ID, PWD) 및 상품정보(ItemAmt, ItemName, ItemCode 등)을 서버로 전송 후 결제창을 호출함.
  1. "다날 인증서버"와 통신하여 가맹점 인증을 받는다.
  2. 인증 성공시, "다날 웹서버"로 FORM 태그를 HTTP POST방식으로 전송하여 결제창을 호출한다.
### 1. 인증 단계 파라미터
#### INPUT
|Field|필수여부|Max(Byte)|비고(예제)
|---|---|---|---
|CP 정보|||
|ID|필수|10|CP ID
|PWD|필수|10|CP PWD
|SUBCP|선택사항| |Sub CPID (사용하는 경우만)
|사용자 정보|||
|USERID|선택사항|60|고객 ID
|상품 정보|||
|ItemCount|필수|-|고정값

#### OUTPUT
Field|필수여부|Max(Byte)|비고(예제)
---|---|---|---
Result|필수|4|결과코드
ErrMsg|필수|256|결과메시지
ServerInfo|필수|128|거래인증 key(거래세션)

### 2. 결제창 호출 단계 파라미터
#### INPUT (CP WEB SERVER → DANAL WEB SERVER)
Field|필수여부|비고(예제)
---|---|---
ServerInfo | 필수 | 다날 인증서버에서 받은 거래 키(128 byte )
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
BgColor | 선택사항 | 결제화면 Color 설정시 사용 <br> 00:블루, 01:남색, 02:노랑, 03:민트, 04:보라, 05:분홍, 06:빨강, 07:주황, 08:청록, 09:초록, 10:회색
Email | 선택사항 | 사용자 E-mail주소 = 결제화면에 표기
ByPassValue | 선택사항 | CPCGI 페이지로 전달될 데이터
UseAuthKey | 선택사항 | 사용자 인증 키 전달 여부 <br> 'Y' 전달 시 사용자 인증 완료 시 전달

#### OUTPUT (DANAL WEB SERVER → CP WEB SERVER)
> DANAL WEB SERVER → CP ( TargetURL )
> - PORT : 443
> - Method : POST
> - CHARACTER SET : EUC-KR


Field|반환조건|비고(예제)
---|---|---
ServerInfo | 무조건 | 다날 인증서버에서 받은 거래 키(128 byte)
PayType | 무조건 | 결제 유형 <br> 0: 일반결제 (월자동결제를 위한 첫결제 포함)
DstAddr | 무조건 | 사용자 전화번호 (전체 번호)
DstAddr | 무조건 | 사용자 전화번호 (첫 3자리 번호)
DstAddr | 무조건 | 사용자 전화번호 (다음 4자리 번호)
DstAddr | 무조건 | 사용자 전화번호 (다음 4자리 번호)
CPName | 무조건 | 결제화면에 표시되는 **가맹점 이름**
ItemName | 무조건 | 결제화면에 표시되는 **상품 이름**
ItemAmt | 무조건 | 결제화면에 표시되는 **결제금액 **
TargetURL | 무조건 | 최종 결제 요청할 CP의 CPCGI FULL URL <br> (https://cp.site.co.kr/xxx/xxx/CPCGI.*)
BackURL | 조건부 | 에러 발생 및 취소 시 이동할 페이지의 FULL URL <br> (https://cp.site.co.kr/xxx/xxx/BackURL.*)
IsUseCI | 조건부 | 가맹점 CI 사용여부
CIURL | 조건부 | 가맹점 CI 이미지 페이지 FULL URL<br>(https://cp.site.co.kr/xxx/xxx/cpci.jpg)<br><br>CI 이미지 사이즈 = 119x47
IsPreOtbill | 조건부 | 자동결제, Authkey 수신 여부(Y/N) <br> (결제화면 내 제공기간 항목에 표기)
IsSubscript | 조건부 | 월 정액 가입 여부(Y/N) <br> (결제화면 내 제공기간 항목에 표기)
IsCharSet | 조건부 | CP 측 웹서버 인코딩 방식
BgColor | 조건부 | 결제화면 Color 설정시 사용 <br> 00:블루, 01:남색, 02:노랑, 03:민트, 04:보라, 05:분홍, 06:빨강, 07:주황, 08:청록, 09:초록, 10:회색
Email | 조건부 | 사용자 E-mail주소 = 결제화면에 표기
ByPassValue | 조건부 | CPCGI 페이지로 전달될 데이터 (있을 시)
AUTHKEY | 조건부 | 사용자 인증 키(UseAuthKey로 요청 시)
- 개인정보 정책에 의해 인증번호, 통신사정보, 잔여한도금액은 전달할 수 없습니다.


## II. CPCGI 페이지 - 결제건 검증, 결제
* 역할 : 결제정보 검증 단계, 결제 단계 ( D, E 단계 )
* 설명 : Order ID 또는 결제금액 등 결제 거래내용에 대한 검증 단계, 그 다음 결제 요청을 보낸다.

#### INPUT ( NCONFIRM, 검증 단계 )
Field|필수여부|Max(Byte)|비고(예제)
---|---|---|---
ServerInfo | 필수 | 128 | 다날 인증서버에서 받은 거래 키(128 byte 고정)
ConfirmOption | 선택사항 | 1 : CPID, ORDERID 한번 더 체크 <br> 0 : 검증 안함(Default)
AMOUNT | 선택사항 | 6 | 결제 금액 비교 시 사용 <br> ConfirmOption 1 인 경우 필수
CPID | 선택사항 | 10 | CPID 비교 시 사용 <br> ConfirmOption 1 인 경우 필수
Command | 필수 | - | "NCONFIRM" - 고정 값
OUTPUTOPTION | 필수 | - | "DEFAULT" - 고정 값
IFVERSION | 필수 | - | "V1.1.2" - 고정 값

#### OUTPUT ( NCONFIRM, 검증 단계 )
Field|반환조건|Max(Byte)|비고(예제)
---|---|---|---
Result | 무조건 | 4 | 결과 코드 (성공 시 0) 
ErrMsg | 무조건 | 255 | 결과 메시지 (성공 시 No Information)
CPID | 조건부 | 10 | ConfirmOption 이 1 인 경우 반환
SUBCP | 조건부 | 10 | SUBCPID 가 없을 경우 'none'
AMOUNT | 조건부 | 6 | ConfirmOption 이 1인 경우 반환
ORDERID | 조건부 | 200 | 있을 시에 반환
TID | 조건부 | 거래번호 ( 년간 Unique ) <br> IFVERSION = V1.1.2 이상인 경우

\* Parameter값 실제 전송형태 예제
> "ServerInfo=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx;Command=NCONFIRM;OUTPUTOPTION=Default"



#### INPUT ( NBILL, 결제 단계 )
Field|필수여부|Max(Byte)|비고(예제)
---|---|---|---
ServerInfo | 필수 | 128 | 다날 인증서버에서 받은 거래 키(128 byte 고정)
BillOption | 선택사항 | 1 | 0 : Default <br> 1 : Authkey반환
Command | 필수 | - | "NCONFIRM" - 고정 값
OUTPUTOPTION | 필수 | - | "DEFAULT" - 고정 값
IFVERSION | 필수 | - | "V1.1.2" - 고정 값

#### OUTPUT ( NBILL, 결제 단계 )
Field|반환조건|Max(Byte)|비고
---|---|---|----
Result | 무조건 | 4 | 결과 코드 (성공 시 0) 
ErrMsg | 무조건 | 255 | 결과 메시지 (성공 시 No Information)
DATE | 무조건 | 14 | 결제 완료 시간
TID | 무조건 | 18 | 거래 번호 (년간 고유)
DNTID | 조건부 | 22 | 거래 번호 (YYYY + TID)
Authkey | 조건부 | 20 | 2회차 이상 결제 혹은 자동결제 등 상황에 사용되는 <br> 사용자 인증 키 (20150701... 같이 연도로 시작)
ORDERID | 조건부 | 200 | 요청시 보낸 가맹점 주문번호 (IFVERSION = V1.1.2 이상인 경우)



### CP 주의 사항
* NBILL 이 성공적으로 수행된 이후 (Result=0), 이용자에게 서비스를 제공한다.
* 결제건의 대사, 취소를 위하여 TID(18byte) 를 저장한다.
* 결제서버로 부터 받은 응답 결과에 대해 성공, 실패 여부 체크 및 데이터 저장을 위한 파싱 과정에서 공백 처리 및 대소문자 구분에 유의한다.
 * 다날에서 제공하는 function.* 파일 속 String -> Array 파싱 함수를 참조하여 사용하는 것을 권장하며, Key=Value 형식으로 구분자로 파싱하여 사용하도록 한다.



## 거래 취소 (BILL_CANCEL)
### 취소 방법
1. 관리자 페이지에서 취소
    * https://partner.danalpay.com 로그인 후, 사용중인 결제 서비스 클릭 후 건별 취소 가능
3. BILL_CANCEL 취소 요청
#### INPUT
Field | 필수여부 | Max(Byte) | 비고(예제)
---|---|---|---
ID | 필수 | 10 | CPID
PWD | 필수 | 10 | CP password
TID | 필수 | 18 | 거래 고유 번호
Command | 필수 | - | "BILL_CANCEL" - 고정 값
OUTPUTOPTION | 필수 | - | "3" - 고정 값

#### OUTPUT
Field|반환조건|Max(Byte)|비고
---|---|---|----
Result | 무조건 | 4 | 결과 코드 (성공 시 0) 
ErrMsg | 무조건 | 255 | 결과 메시지 (성공 시 No Information)
DATE | 무조건 | 14 | 결제 완료 시간
TID | 무조건 | 18 | 거래 번호 (년간 고유)


---------------------------
## 그 외 참고사항

#### Java 연동 시
* 모든 설정 파일 (java : info.properties / 그 외 : teledit.conf) 의 기본 경로는 /etc/
* java 설정 파일은 JVM이 로딩되는 엔트리 포인트에 있어야 하며, 설정파일을 읽지 못하는 경우 클라이언트에 설정되어있는 default 값을 이용한다.
* java 설정파일 위치 별도 지정 : Output = new TeleditClient("./home/test/info/").SClient( Input );
* log 파일 추가시, info.properties 옵션값으로 설정 가능하다.
* Logdir - log 디렉터리 지정, LogIF - 로그 기능 사용 여부 (y, n 값)

#### Java 외 다른 형식으로 연동 시 (PHP, Nodejs, Python 등)
* 에러코드 127 은 SClient 실행파일의 권한 / 디렉터리 등 문제이다.
