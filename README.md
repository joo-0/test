**목차**   
- [개요](#개요)
    - [대상](#대상)
    - [용어](#용어)
- [온라인 프로그램 개발 따라하기](#온라인-프로그램-개발-따라하기)
    - [입출력전문 VO 클래스 작성](#1-입출력전문-VO-클래스-작성)
        - [입력전문 VO 작성](#11-입력전문-vo-작성)
        - [출력전문 VO 작성](#)
    - [입출력전문 등록](#)
    - [SQL 쿼리 작성](#)
    - [자바 프로그램 작성](#)
        - [Entity VO 작성](#)
        - [EBC 작성](#)
        - [PBC 작성](#)
    - [](#)
 ...
 
# 개요
본 문서는 프로젝트 차세대 시스템 업무계의 온라인 프로그램 개발을 위한 가이드 문서이다.

## 대상
업무계 시스템 온라인 서버 프로그램 개발자

## 용어
|용어|설명|비고|
|:---:|:---|:---:|
|`서비스`|시스템에서 외부로 오픈 된 거래처리 단위를 서비스라 한다. PBC (Process Business Component)의 Method와 매핑됨.|
|`IN/OUT SPEC`|`IN SPEC – 서비스의 입력 파라미터 Layout, OUT SPEC - 서비스의 출력 파라미터 Layout||
|`전문`|시스템간 주고 받는 표준형식의 데이터 (Fixed Length 형태의 String)||
|`VO`|전달된 전문은 업무계 시스템의 DevOnFrame에서 인식할 수 있는 형식의 객체로 변환한다.  java의 setter/getter객체 클래스이다.||

# 온라인 프로그램 개발 따라하기
본 장은 온라인 개발을 시작하는 개발자가 온라인 프로그램의 개발절차를 이해하기 쉽도록 따라할 수 있는 간단한 샘플프로그램 개발절차이다.
devonframe enterprise prototype sample의 계좌조회 구현 내용을 바탕으로 설명되어있다.

## 1. 입출력전문 VO 클래스 작성 
입출력전문 정보를 기반으로 DevOnFrame에서 사용하기 위한 전문 VO 클래스를 먼저 생성해야 한다.
계좌조회에 사용되는 입출력전문 정보는 아래와 같다.

- 입력전문
    - 계좌번호 (문자열, 6자)
    - 고객번호 (문자열, 10자)
- 출력전문
    - 고객계좌정보 (단건, 61자)
        - 고객ID (문자열, 10자)
        - 계좌개설점 (실수, 1자)
        - 계좌번호 (문자열, 6자)
        - 비밀번호 (문자열, 4자)
        - 잔액 (실수, 20자)
        - 고객이름 (문자열, 20자)
    - 계좌거래정보 건수 (실수, 5자)
    - 계좌거래정보 (다건, 46자)
        - 계좌번호 (문자열, 6자)
        - 거래일자 (문자열, 8자)
        - 거래시각 (문자열, 6자)
        - 거래금액 (실수, 20자)
        - 거래유형명 (문자열, 6자)
```
// 입력전문 예시
1000011         

// 출력전문 예시
CID_00001 3100001123400000000000082400000CNM_1               00030100001              00000000000000000000입금100001              00000000000000000000입금100001              00000000000000000000입금100001              00000000000000000000입금100001              00000000000000000000입금100001              00000000000000000000입금100001              00000000000000000000입금100001              00000000000000000000입금100001              00000000000000000000입금100001              00000000000000000000입금100001              00000000000000000000입금100001              00000000000000000000입금100001              00000000000000000000입금100001              00000000000000000000입금100001              00000000000000000000입금100001              00000000000000000000입금100001              00000000000000000000입금1000012018120311080900000000000000100000출금1000012018120311054000000000000000100000출금1000012018112914593500000000000000100000출금1000012018112914553300000000000000100000출금1000012018111417390400000000000000100000출금10000120180305      00000000000000100000출금10000120180227      00000000000000100000출금10000120180227      00000000000000100000출금10000120180227      00000000000000100000출금10000120180227      00000000000000100000출금10000120180227      00000000000000100000출금10000120180227      00000000000000100000출금10000120180227      00000000000000100000출금
```

### 1.1. 입력전문 VO 작성
입력전문 정보를 기반으로 입력 VO 클래스를 작성한다.
```java
public class AccountInfoInputDto { // 계좌조회 입력 VO
    private String cusNo;
    private String accNum;

    public String getCusNo() {
        return cusNo;
    }

    public void setCusNo(String cusNo) {
        this.cusNo = cusNo;
    }

    public String getAccNum() {
        return accNum;
    }

    public void setAccNum(String accNum) {
        this.accNum = accNum;
    }
}
```
### 1.2. 출력전문 VO 작성
출력전문 정보를 바탕으로 출력 VO 클래스를 작성한다.
출력전문이 단건일 경우 1)번처럼 작성하면 되지만, 다건일 경우에는 아래처럼 다건 VO와 해당 다건 VO의 count 정보를 함께 정의해야 한다.
 - List 와 List 건수 set/get 하기 위한 출력전문 VO 객체 
 - 다건 group(accountTrxInfoDto)과 groupCnt(accountTrxInfoDtoCnt)는 출력전문에 정의되어야 한다. (2. 입출력전문 등록 확인)	
```java
public class AccountInfoOutputDto { // 계좌조회 출력 VO
    private CustomerAccountDto cusAccountInfoDto; // 고객계좌 VO (단건)
    private List<AccountTrxInfoDto> accountTrxInfoDto; // 계좌 거래내역 VO (다건)
    private BigDecimal accountTrxInfoDtoCnt; // 다건인 계좌 거래내역 VO의 총 건수

    public CustomerAccountDto getCusAccountInfoDto() {
        return cusAccountInfoDto;
    }

    public void setCusAccountInfoDto(CustomerAccountDto cusAccountInfoDto) {
        this.cusAccountInfoDto = cusAccountInfoDto;
    }

    public List<AccountTrxInfoDto> getAccountTrxInfoDto() {
        return accountTrxInfoDto;
    }

    public void setAccountTrxInfoDto(List<AccountTrxInfoDto> accountTrxInfoDto) {
        this.accountTrxInfoDto = accountTrxInfoDto;
    }

    public BigDecimal getAccountTrxInfoDtoCnt() {
        return accountTrxInfoDtoCnt;
    }

    public void setAccountTrxInfoDtoCnt(BigDecimal accountTrxInfoDtoCnt) {
        this.accountTrxInfoDtoCnt = accountTrxInfoDtoCnt;
    }
    ... 중략
}
```
## 2. 입출력전문 등록 
> 뭘로할건지 결정된 후 작성(DevonFrame Studio??? 새운영관리???)

MIO 전문 생성 도구를 이용하여 전문을 생성하며 등록된 전문은 DevOn 운영관리 사이트에서도 확인할 수도 있다.

### 2.1. MIO 프로젝트 생성
MIO 프로젝트가 없는 경우는 1)처럼 MIO를 wizard에서 MIO프로젝트를 생성해야 한다.

## 3. SQL 쿼리 작성
계좌조회의 출력전문에 담긴 정보를 조회할 수 있는 SQL 쿼리를 작성한다.
 - 고객 테이블: CUS_MST
 - 계좌 테이블: RCP_MST
 - 계좌내역 테이블: RCP_LIST

## 4. 자바 프로그램 작성
### 4.1. Entity VO 작성
Entity VO는 DB의 테이블과 매핑되는 클래스로 DB 테이블의 각 컬럼이 필드로 정의된다.
계좌조회에는 고객, 계좌, 계좌내역 Entity가 사용된다.
아래는 고객 테이블과 해당 테이블에 매핑되는 고객 Entity 샘플 이다. 

- 고객 테이블: CUS_MST

 |컬럼 명|속성|설명|
 |:---|:---|:---|
 |CUS_NO|VARCHAR(10)|고객번호|
 |CUS_NM|VARCHAR(20)|고객이름|
 |CUS_ID|VARCHAR(10)|고객ID|
 
- 고객 Entity: CustomerEntity
 ```java
public class CustomerEntity {
    private String cusNo;
    private String cusId;
    private String cusNm;

    public String getCusNo() {
        return cusNo;
    }

    public void setCusNo(String cusNo) {
        this.cusNo = cusNo;
    }

    public String getCusId() {
        return cusId;
    }

    public void setCusId(String cusId) {
        this.cusId = cusId;
    }

    public String getCusNm() {
        return cusNm;
    }

    public void setCusNm(String cusNm) {
        this.cusNm = cusNm;
    }
}
```
다른 Entity들도 CustomerEntity와 같은 방법으로 작성한다.

### 4.2. EBC 작성
EBC는 비즈니스 정보를 DB에 저장하거나 DB에 저장되어 있는 비즈니스 정보를 수정, 삭제, 조회하는 작업을 처리하는 컴포넌트이다.
주입받은 CommonDao를 이용하여 CRUD가 일어나며, 앞서 작성한 SQL 쿼리가 사용된다. 아래는 고객 EBC 샘플이다. 
```java
@Component
public class CustomerEbc {

    @Resource(name = "commonDao")
    private CommonDao commonDao;
    
    public CustomerEntity retrieveCustomer(CustomerEntity input){
        return commonDao.select("CustomerAccount.retrieveCustomer", input);
    }
}
```
EBC 내부에서 CommonDao 호출 시 앞에서 작성한 SQL ID를 첫번째 파라메터로 넘겨준다. 
이 때, 해당 SQL ID 에는 namespace 가 앞에 붙어야 함에 유의한다.

|Namespace|SQL ID|Dao 호출 시|
|:---|:---|:---|
|CustomerAccount|retrieveCustomer|CustomerAccount.retrieveCustomer|

### 4.3. PBC 작성
PBC는 비즈니스 프로세스를 구현하는 책임을 담당하는 컴포넌트로서 
주로 하나 이상의 엔터티 컴포넌트와 공통 프로세스 컴포넌트들의 상호작용을 통하여 비즈니스 프로세스를 처리한다.
아래는 계좌조회 PBC 샘플의 일부 이다.
 - PBC Class에 `@Component` 선언하여 Bean으로 등록
 - EBC와 CPBC를 주입받기 위해 `@Resource` 선언하며, Bean ID는 FullyQualifiedBeanName 적용 (예: `prototype.neoent.customerAccount.ebi.AccountTrxEbc`)
 - 고객계좌공통 CPBC와 계좌거래내역 EBC를 호출하여 고객계좌정보와 계좌거래내역을 조회한 뒤, 출력전문 VO에 해당 결과들을 넣어 리턴함
```java
@Component
public class AccountInfoServicePbc {

    // 계좌거래내역 EBC 주입
	@Resource(name = "prototype.neoent.customerAccount.ebi.AccountTrxEbc")
	private AccountTrxEbc accountTrxEbc;

	// 고객계좌공통 CPBC 주입
	@Resource(name = "prototype.neoent.customerAccount.cpbi.CustomerAccountCommonCpbc")
	CustomerAccountCommonCpbc customerAccountCommonCpbc;
	
	public AccountInfoOutputDto serviceAccountInfoInq(AccountInfoInputDto iAccountInfoInqService) throws BusinessException {
	    
            ... (앞부분 생략)

            // =============================================================================
            // ######### 고객계좌공통 조회
            // =============================================================================

            // 고객계좌조회 조건 설정
            iCustomerAccountCommonInq.setCusNo(iAccountInfoInqService.getCusNo());
            iCustomerAccountCommonInq.setAccNum(iAccountInfoInqService.getAccNum());
    
            // 고객계좌 CPBC 호출 & 조회
            rCustomerAccountCommonInq = customerAccountCommonCpbc
                    .retrieveCustomerAccountCommon(iCustomerAccountCommonInq); 
    
            // NOT FOUND 처리
            if (rCustomerAccountCommonInq == null  || "".equals(rCustomerAccountCommonInq.getAccNum())) {
                {
                    throw new BusinessException("EM00012", new StackTraceExceptionOptionalInfo("계좌조회 에러 : 계좌번호가 존재하지 않습니다."));
                }
            }
    
            // =============================================================================
            // ######### 계좌거래내역 조회
            // =============================================================================
           
            // 계좌거래내역 조건 설정
            iAccountTrxListInq.setAccNum(iAccountInfoInqService.getAccNum());

            // 계좌거래내역 EBC 호출
            rAccountTrxListInq = accountTrxEbc
                    .retrieveAccountTrxList(iAccountTrxListInq);
    
            // =============================================================================
            // ######### 출력전문 VO 세팅
            // =============================================================================
            
            // 고객계좌정보(단건) 세팅
            CustomerAccountDto tCusAccountInfo = new CustomerAccountDto(); 
            tCusAccountInfo.setCusId(rCustomerAccountCommonInq.getCusId());
            tCusAccountInfo.setOpBrch(rCustomerAccountCommonInq.getOpBrch());
            tCusAccountInfo.setAccNum(rCustomerAccountCommonInq.getAccNum());
            tCusAccountInfo.setAccPw(rCustomerAccountCommonInq.getAccPw());
            tCusAccountInfo.setAmt(rCustomerAccountCommonInq.getAmt());
            tCusAccountInfo.setCusNm(rCustomerAccountCommonInq.getCusNm());
    
            // 계좌거래내역(다건) 세팅
            List<AccountInfoOutputDto.AccountTrxInfoDto> tAccountTrxInfo = new ArrayList<>(); 
            for (AccountTrxEntity ta_rAccountTrxListInq : rAccountTrxListInq) { 
                AccountInfoOutputDto.AccountTrxInfoDto ta__tAccountTrxInfo = new AccountInfoOutputDto.AccountTrxInfoDto();
                ta__tAccountTrxInfo.setAccNum(ta_rAccountTrxListInq.getAccNum()); 
                ta__tAccountTrxInfo.setTrxAmt(ta_rAccountTrxListInq.getTrxAmt()); 
                ta__tAccountTrxInfo.setTrxDate(ta_rAccountTrxListInq.getTrxDate()); 
                ta__tAccountTrxInfo.setTrxTime(ta_rAccountTrxListInq.getTrxTime()); 
                ta__tAccountTrxInfo.setTrxTypeNm(ta_rAccountTrxListInq.getTrxTypeNm()); 
                tAccountTrxInfo.add(ta__tAccountTrxInfo);
            }
            
            // 출력전문 VO 세팅
            rAccountInfoInqService.setCusAccountInfoDto(tCusAccountInfo);
            rAccountInfoInqService.setAccountTrxInfoDto(tAccountTrxInfo);
            rAccountInfoInqService.setAccountTrxInfoDtoCnt(new BigDecimal(tAccountTrxInfo.size()));
    
            // 출력전문 VO 리턴
            rdata = rAccountInfoInqService; 
            return rdata;
	}
}
```
### 4.4. CPBC 작성

## 5. 서비스 등록
DevOn 운영관리 사이트의 서비스관리 > 파라미터관리 > 서비스파라미터관리 화면에서 서비스에 대한 정보를 등록한다.
### 5.1. 마스터정보 등록
신규 버튼을 클릭하여 '마스터 정보' 탭에서 표준명명규칙에 맞는 서비스ID, 서비스명, 업무코드, 상태코드(정상) 을 등록한다.
 ![ex_screenshot](./image/online/3.1.마스터정보등록.jpg)
### 5.2. 기본정보 등록
'기본정보' 탭에서 앞의 [4.3번](#4.3.-PBC-작성)에서 구현했던 PBC 클래스 및 메소드를 등록한다.   
SPEC ID 에는 [2번](#2.-입출력전문-등록)에서 등록했던 입출력전문 스펙을 입력하고, INPUT/OUTPUT VO에는 [1번](#1.-입출력전문-VO-클래스-작성)에서 작성했던 클래스명을 입력한다.
다른 탭의 항목들은 default 상태 그대로 둔 뒤 저장한다.
![ex_screenshot](./image/online/3.2.기본정보등록.jpg)

## 6. 거래테스트
DevonFrame Studio 의 거래테스트도구를 이용하여 거래테스트를 수행한다. 
거래테스트를 로컬에서 수행하고자 한다면, 다음과 같이 로컬테스트서버를 먼저 구동시켜야 한다. 
(prototype 샘플을 테스트 하기 위해서 devon-enterprise-prototype-sample 온라인 테스트 서버(Tomcat v7.0)을 구동시켜야 한다.

#거래 유형별 샘플 위치
