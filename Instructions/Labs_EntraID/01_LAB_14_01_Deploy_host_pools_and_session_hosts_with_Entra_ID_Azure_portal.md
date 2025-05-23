---
lab:
  title: '랩: Azure Portal을 사용하여 호스트 풀 및 세션 호스트 배포(Entra ID)'
  module: 'Module 1.4: Implement host pools and session hosts'
---

# 랩 - Azure Portal을 사용하여 호스트 풀 및 세션 호스트 배포(Entra ID)
# 학생용 랩 매뉴얼

## 랩 종속성

- 이 랩에서 사용할 Azure 구독
- 이 랩에서 사용할 Azure 구독에 대한 Owner 역할과, 해당 Azure 구독에 연결된 Microsoft Entra 테넌트에 디바이스를 조인하기에 충분한 사용 권한이 있는 Microsoft Entra 계정.

## 예상 소요 시간

60분

## 랩 시나리오

Microsoft Azure 구독이 있어야 합니다. Microsoft Entra 조인 세션 호스트를 사용하는 Azure Virtual Desktop 환경을 배포해야 합니다.

## 목표
  
이 랩을 완료하면 다음을 수행할 수 있습니다.

- Azure Virtual Desktop에 조인된 Microsoft Entra 세션 호스트 배포

## 랩 파일

- None

## 지침

### 연습 1: Microsoft Entra 조인 세션 호스트를 사용하여 Azure Virtual Desktop 환경 구현
  
이 연습의 주요 작업은 다음과 같습니다.

1. Azure Virtual Desktop 호스트 풀 배포용 Azure 구독 준비
1. Azure Virtual Desktop 호스트 풀 배포
1. Azure Virtual Desktop 애플리케이션 그룹 구성
1. Azure Virtual Desktop 작업 영역 만들기
1. Azure Virtual Desktop 호스트 풀에 대한 액세스 허용

#### 작업 1: Azure Virtual Desktop 호스트 풀 배포용 Azure 구독 준비

1. 랩 컴퓨터에서 웹 브라우저를 시작하고, Azure Portal([https://portal.azure.com](https://portal.azure.com))로 이동하여 이 랩에서 사용할 구독에서 소유자 역할이 있는 사용자 계정의 자격 증명을 입력하여 로그인합니다.

    > **참고**: 랩 세션 창의 오른쪽에 있는 리소스 탭에 나열된 `User1-` 계정의 자격 증명을 사용합니다.

1. Azure Portal의 Azure Cloud Shell에서 PowerShell 세션을 시작합니다.

    > **참고**: 메시지가 표시되면 **시작** 창의 **구독** 드롭다운 목록에서 이 랩에서 사용 중인 Azure 구독의 이름을 선택한 다음 **적용**을 선택합니다.

1. Azure Cloud Shell 창의 PowerShell 세션에서 다음 명령을 실행하여 **Microsoft.DesktopVirtualization** 리소스 공급자를 등록합니다.

    ```powershell
    Register-AzResourceProvider -ProviderNamespace Microsoft.DesktopVirtualization
    ```

    > **참고**: 등록이 완료되기를 기다리지 마세요. 이 과정은 몇 분 정도 걸릴 수 있습니다.

1. Cloud Shell 창을 닫습니다.
1. Azure Portal이 표시된 웹 브라우저에서 **가상 네트워크**를 검색하여 선택하고 **가상 네트워크** 페이지에서 **+ 만들기**를 선택합니다.
1. **가상 네트워크 만들기** 페이지의 **기본** 탭에서 다음 설정을 지정하고 **다음**을 선택합니다.

    |설정|값|
    |---|---|
    |구독|이 랩에서 사용 중인 Azure 구독의 이름|
    |Resource group|새 리소스 그룹의 이름 **az140-11e-RG**|
    |가상 네트워크 이름|**az140-vnet11e**|
    |지역|Azure Virtual Desktop 환경을 배포하려는 Azure 지역의 이름|

1. **보안** 탭에서 기본 설정을 수락하고 **다음**을 선택합니다.
1. **IP 주소** 탭에서 다음 설정을 적용합니다(필요한 경우 기본값 수정).

    |설정|값|
    |---|---|
    |IP 주소 공간|**10.20.0.0/16**|

1. **기본** 서브넷 항목 옆에 있는 편집(연필) 아이콘을 선택하고 **편집** 창에서 다음 설정을 지정하고(다른 설정을 기존 값으로 유지) **저장**을 선택합니다.

    |설정|값|
    |---|---|
    |속성|**hp1-Subnet**|
    |시작 주소|**10.20.1.0**|
    |프라이빗 서브넷 활성화(기본 아웃바운드 액세스 없음)|사용 안 함|

1. **IP 주소** 탭으로 돌아가서 **검토 + 만들기**를 선택한 다음 **검토 + 만들기** 탭에서 **만들기**를 선택합니다.

    > **참고**: 프로비저닝 프로세스가 완료될 때까지 기다리지 마십시오. 이 단계는 대개 1분 이내에 완료됩니다.

1. Azure Portal이 표시된 웹 브라우저에서 **Microsoft Entra ID**를 검색하여 선택합니다.
1. 구독과 연결된 Microsoft Entra 테넌트 **개요** 페이지의 세로 탐색 메뉴의 **관리** 섹션에서 **사용자**를 선택합니다.
1. **사용자** 페이지의 **검색** 텍스트 상자에 랩 세션 창의 오른쪽에 있는 리소스 탭에 나열된 `User1-` 계정의 이름을 입력합니다.
1. 검색 결과 목록에서 일치하는 이름을 가진 사용자 계정 항목을 선택합니다.
1. 사용자 계정의 속성을 표시하는 페이지의 세로 탐색 메뉴의 **관리** 섹션에서 **그룹**을 선택합니다.
1. **그룹** 페이지에서 **AVD-DAG** 접두사로 시작하는 그룹의 이름(이 랩의 뒷부분에서 필요)을 기록합니다.
1. **사용자** 페이지로 돌아가서 **검색** 텍스트 상자에 랩 세션 창의 오른쪽에 있는 리소스 탭에 나열된 `User2-` 계정의 이름을 입력합니다.
1. 검색 결과 목록에서 일치하는 이름을 가진 사용자 계정 항목을 선택합니다.
1. 사용자 계정의 속성을 표시하는 페이지의 세로 탐색 메뉴의 **관리** 섹션에서 **그룹**을 선택합니다.
1. **그룹** 페이지에서 **AVD-RemoteApp** 접두사로 시작하는 그룹의 이름(이 랩의 뒷부분에서 필요)을 기록합니다.

#### 작업 2: Azure Virtual Desktop 호스트 풀 배포

1. 랩 컴퓨터의 Azure Portal이 표시된 웹 브라우저 창에서 **Azure Virtual Desktop**을 검색하여 선택하고 **Azure Virtual Desktop** 페이지의 세로 탐색 메뉴의 **관리** 섹션에서 **호스트 풀**을 선택하고 **Azure Virtual Desktop \| 호스트 풀** 페이지에서 **+ 만들기**를 선택합니다. 
1. **호스트 풀 만들기** 페이지의 **기본** 탭에서 다음 설정을 지정하고 **다음: 세션 호스트 >** 을 선택합니다(다른 설정은 기본값 유지).

    |설정|값|
    |---|---|
    |구독|이 랩에서 사용 중인 Azure 구독의 이름|
    |Resource group|새 리소스 그룹 **az140-21e-RG**의 이름|
    |호스트 풀 이름|**az140-21-hp1**|
    |위치|Azure Virtual Desktop 환경을 배포하려는 Azure 지역의 이름|
    |유효성 검사 환경|**문제**|
    |기본 설정 앱 그룹 유형|**데스크톱**|
    |호스트 풀 유형|**풀링됨**|
    |세션 호스트 구성 만들기|**No**|
    |부하 분산 알고리즘|**너비 우선**|

    > **참고**: 폭 우선 부하 분산 알고리즘을 사용하는 경우 최대 세션 제한 매개 변수는 선택 사항입니다.

1. **호스트 풀 만들기** 페이지의 **세션 호스트** 탭에서 다음 설정을 지정하고 **다음: 작업 영역 >** 을 선택합니다(다른 모든 설정은 기본값 유지).

    > **참고**: **이름 접두사** 값을 설정할 때 랩 세션 창의 오른쪽에 있는 리소스 탭으로 전환하고 *User1*과 *@* 문자 사이의 문자열을 식별합니다. 이 문자열을 사용하여 *random* 자리 표시자를 바꿉니다.

    |설정|값|
    |---|---|
    |가상 컴퓨터 추가|**예**|
    |Resource group|**호스트 풀과 같은 그룹으로 기본 지정됨**|
    |이름 접두사|**sh**-*random*|
    |가상 머신 유형|**Azure 가상 머신**|
    |가상 머신 위치|Azure Virtual Desktop 환경을 배포하려는 Azure 지역의 이름|
    |가용성 옵션|**인프라 중복 필요 없음**|
    |보안 유형|**신뢰할 수 있는 시작 가상 머신**|
    |이미지|**Windows 11 Enterprise 다중 세션, 버전 23H2 + Microsoft 365 Apps**|
    |가상 머신 크기|**표준 DC2s_v3**|
    |VM 수|**2**|
    |OS 디스크 유형|**표준 SSD**|
    |OS 디스크 크기|**기본 크기(128GiB)**|
    |부팅 진단|**관리형 스토리지 계정으로 활성화(권장)**|
    |가상 네트워크|**az140-vnet11e**|
    |서브넷|**hp1-Subnet**|
    |네트워크 보안 그룹|**기본**|
    |퍼블릭 인바운드 포트|**문제**|
    |조인할 디렉터리를 선택합니다.|**Microsoft Entra ID**|
    |Intune에 VM 등록|**No**|
    |사용자 이름|**Student**|
    |암호|기본 제공 관리자 계정의 암호로 사용될 충분히 복잡한 문자열|
    |암호 확인|이전에 지정한 것과 동일한 문자열|

    > **참고**: 암호는 길이가 12자 이상이어야 하며 소문자, 대문자, 숫자 및 특수 문자의 조합으로 구성됩니다. 자세한 내용은 [Azure VM을 만들 때의 암호 요구 사항](https://learn.microsoft.com/en-us/azure/virtual-machines/windows/faq#what-are-the-password-requirements-when-creating-a-vm-)에 대한 정보를 참조하세요.

1. **호스트 풀 만들기** 페이지의 **작업 영역** 탭에서 다음 설정을 확인하고 **검토 + 만들기**를 선택합니다.

    |설정|값|
    |---|---|
    |데스크톱 앱 그룹 등록|**No**|

1. **호스트 풀 만들기** 페이지의 **검토 + 만들기** 탭에서 **만들기**를 선택합니다.

    > **참고**: 배포가 완료될 때까지 기다립니다. 20분 정도 걸릴 수 있습니다.

#### 작업 3: Azure Virtual Desktop 애플리케이션 그룹 구성

1. 랩 컴퓨터의 Azure Portal을 표시하는 웹 브라우저에서 **Azure Virtual Desktop**을 검색하여 선택하고, **Azure Virtual Desktop** 페이지에서 **애플리케이션 그룹**을 선택합니다.
1. **Azure Virtual Desktop \| 애플리케이션 그룹** 페이지에서 자동 생성된 기존 **az140-21-hp1-DAG** 데스크톱 애플리케이션 그룹을 찾아서 선택합니다. 
1. **az140-21-hp1-DAG** 페이지에서 세로 탐색 메뉴의 **관리** 섹션에서 **할당**을 선택합니다.
1. **az140-21-hp1-DAG \| 할당** 페이지에서 **+ 추가**를 선택합니다.
1. **Microsoft Entra 사용자 또는 사용자 그룹 선택** 페이지에서 **그룹**을 선택하고 검색 상자에 이 연습의 첫 번째 작업에서 식별한 **AVD-DAG** 그룹의 전체 이름을 입력하고, 그룹 이름 옆에 있는 체크박스를 선택하고 **선택**을 클릭합니다.
1. **Azure Virtual Desktop \| 애플리케이션 그룹** 페이지로 다시 이동하여 **+ 만들기**를 선택합니다. 
1. **애플리케이션 그룹 만들기** 페이지의 **기본** 탭에서 다음 설정을 지정하고 **다음: 애플리케이션 >** 을 선택합니다.

    |설정|값|
    |---|---|
    |구독|이 랩에서 사용 중인 Azure 구독의 이름|
    |Resource group|**az140-21e-RG**|
    |호스트 풀|**az140-21-hp1**|
    |애플리케이션 그룹 종류|**원격 앱**|
    |애플리케이션 그룹 이름|**az140-21-hp1-Office365-RAG**|

1. **애플리케이션 그룹 만들기** 페이지의 **애플리케이션** 탭에서 **+ 애플리케이션 추가**를 선택합니다.
1. **애플리케이션 추가** 페이지에서 다음 설정을 지정하고 **검토 + 추가**를 선택한 다음, **추가**를 선택합니다.

    |설정|값|
    |---|---|
    |애플리케이션 소스|**시작 메뉴**|
    |애플리케이션|**Word**|
    |표시 이름|**Microsoft Word**|
    |설명|**Microsoft Word**|
    |명령줄 필요|**No**|

1. **애플리케이션 그룹 만들기** 페이지의 **애플리케이션** 탭으로 돌아가서 **+ 애플리케이션 추가**를 선택합니다.
1. **애플리케이션 추가** 페이지에서 다음 설정을 지정하고 **검토 + 추가**를 선택한 다음, **추가**를 선택합니다.

    |설정|값|
    |---|---|
    |애플리케이션 소스|**시작 메뉴**|
    |애플리케이션|**Excel**|
    |표시 이름|**Microsoft Excel**|
    |설명|**Microsoft Excel**|
    |명령줄 필요|**No**|

1. **애플리케이션 그룹 만들기** 페이지의 **애플리케이션** 탭으로 돌아가서 **+ 애플리케이션 추가**를 선택합니다.
1. **애플리케이션 추가** 페이지에서 다음 설정을 지정하고 **검토 + 추가**를 선택한 다음, **추가**를 선택합니다.

    |설정|값|
    |---|---|
    |애플리케이션 소스|**시작 메뉴**|
    |애플리케이션|**PowerPoint**|
    |표시 이름|**Microsoft PowerPoint**|
    |설명|**Microsoft PowerPoint**|
    |명령줄 필요|**No**|

1. **애플리케이션 그룹 만들기** 블레이드의 **애플리케이션** 탭으로 돌아가서 **다음: 할당 >** 을 선택합니다.
1. **애플리케이션 그룹 만들기** 페이지의 **할당** 탭에서 **+ Microsoft Entra 사용자 또는 사용자 그룹 추가**를 선택합니다.
1. **Microsoft Entra 사용자 또는 사용자 그룹 선택** 페이지에서 **그룹**을 선택하고, 이 연습의 첫 번째 작업에서 식별한 **AVD-RemoteApp** 그룹의 전체 이름을 입력하고, 그룹 이름 옆에 있는 체크박스를 선택하고, **선택**을 클릭합니다.
1. **애플리케이션 그룹 만들기** 페이지의 **할당** 탭으로 돌아와 **다음: 작업 영역 >** 을 선택합니다.
1. **작업 영역 만들기** 페이지의 **작업 영역** 탭에서 다음 설정을 지정하고 **검토 + 만들기**를 선택합니다.

    |설정|값|
    |---|---|
    |애플리케이션 그룹 등록|**No**|

1. **애플리케이션 그룹 만들기** 페이지의 **검토 + 만들기** 탭에서 **만들기**를 선택합니다.

    > **참고**: 애플리케이션 그룹이 만들어질 때까지 기다립니다. 이는 1분 이내에 완료됩니다. 

    > **참고**: 다음으로는 파일 경로를 기준으로 하여 애플리케이션 그룹을 애플리케이션 원본으로 만듭니다.

1. Azure Portal을 표시하는 웹 브라우저에서 **Azure Virtual Desktop**을 검색하여 선택하고, **Azure Virtual Desktop** 페이지에서 **애플리케이션 그룹**을 선택합니다.
1. **Azure Virtual Desktop \| 애플리케이션 그룹** 페이지에서 **+ 만들기**를 선택합니다. 
1. **애플리케이션 그룹 만들기** 페이지의 **기본** 탭에서 다음 설정을 지정하고 **다음: 애플리케이션 >** 을 선택합니다.

    |설정|값|
    |---|---|
    |구독|이 랩에서 사용 중인 Azure 구독의 이름|
    |Resource group|**az140-21e-RG**|
    |호스트 풀|**az140-21-hp1**|
    |애플리케이션 그룹 종류|**RemoteApp**|
    |애플리케이션 그룹 이름|**az140-21-hp1-Utilities-RAG**|

1. **애플리케이션 그룹 만들기** 페이지의 **애플리케이션** 탭에서 **+ 애플리케이션 추가**를 선택합니다.
1. **애플리케이션 추가** 페이지의 **기본 사항** 탭에서 다음 설정을 지정하고 **다음**을 선택합니다.

    |설정|값|
    |---|---|
    |애플리케이션 소스|**파일 경로**|
    |Application path|**C:\Windows\system32\cmd.exe**|
    |애플리케이션 식별자|**명령 프롬프트**|
    |표시 이름|**명령 프롬프트**|
    |설명|**Windows 명령 프롬프트**|
    |명령줄 필요|**문제**|

1. **아이콘** 탭에서 다음 설정을 지정하고 **검토 + 추가**를 선택한 다음, **추가**를 선택합니다.

    |설정|값|
    |---|---|
    |아이콘 경로|**C:\Windows\system32\cmd.exe**|
    |아이콘 인덱스|0|

1. **애플리케이션 그룹 만들기** 블레이드의 **애플리케이션** 탭으로 돌아가서 **다음: 할당 >** 을 선택합니다.
1. **애플리케이션 그룹 만들기** 페이지의 **할당** 탭에서 **+ Microsoft Entra 사용자 또는 사용자 그룹 추가**를 선택합니다.
1. **Microsoft Entra 사용자 또는 사용자 그룹 선택** 페이지에서 **그룹**을 선택하고, 이 연습의 첫 번째 작업에서 식별한 **AVD-RemoteApp** 그룹의 전체 이름을 입력하고, 그룹 이름 옆에 있는 체크박스를 선택하고, **선택**을 클릭합니다.
1. **애플리케이션 그룹 만들기** 페이지의 **할당** 탭으로 돌아와 **다음: 작업 영역 >** 을 선택합니다.
1. **작업 영역 만들기** 페이지의 **작업 영역** 탭에서 다음 설정을 지정하고 **검토 + 만들기**를 선택합니다.

    |설정|값|
    |---|---|
    |애플리케이션 그룹 등록|**No**|

1. **애플리케이션 그룹 만들기** 페이지의 **검토 + 만들기** 탭에서 **만들기**를 선택합니다.

    > **참고**: 애플리케이션 그룹이 만들어질 때까지 기다립니다. 이는 1분 이내에 완료됩니다. 

#### 작업 4: Azure Virtual Desktop 작업 영역 만들기

1. 랩 컴퓨터의 Azure Portal이 표시된 웹 브라우저에서 **Azure Virtual Desktop**을 검색하여 선택하고, **Azure Virtual Desktop** 페이지에서 **작업 영역**을 선택합니다.
1. **Azure Virtual Desktop \| 작업 영역** 페이지에서 **+ 만들기**를 선택합니다. 
1. **작업 영역 만들기** 페이지의 **기본 사항** 탭에서 다음 설정을 지정하고 **다음: 애플리케이션 그룹 >** 을 선택합니다.

    |설정|값|
    |---|---|
    |구독|이 랩에서 사용 중인 Azure 구독의 이름|
    |Resource group|**az140-21e-RG**|
    |작업 영역 이름|**az140-21-ws1**|
    |식별 이름|**az140-21-ws1**|
    |위치|이 랩의 첫 번째 연습에서 리소스를 배포한 Azure 지역 또는 이와 가까운 지역의 이름|

1. **작업 영역 만들기** 페이지의 **애플리케이션 그룹** 탭에서 다음 설정을 지정합니다.

    |설정|값|
    |---|---|
    |애플리케이션 그룹 등록|**Yes**|

1. **작업 영역 만들기** 페이지의 **작업 영역** 탭에서 **+ 애플리케이션 그룹 등록**을 선택합니다.
1. **애플리케이션 그룹 추가** 페이지에서 **az140-21-hp1-DAG**, **az140-21-hp1-Office365-RAG**, 및 **az140-21-hp1-Utilities-RAG** 항목 옆의 더하기 기호를 선택하고 **선택**을 클릭합니다. 
1. **작업 영역** 만들기 페이지의 **애플리케이션 그룹** 탭으로 돌아가서 **검토 + 만들기**를 선택합니다.
1. **작업 영역 만들기** 페이지의 **검토 + 만들기** 탭에서 **만들기**를 선택합니다.

#### 작업 5: Azure Virtual Desktop 호스트 풀에 대한 액세스 허용

> **참고**: Microsoft Entra 조인 세션 호스트를 사용하는 경우 Azure Virtual Desktop 사용자 및 관리자에게 적절한 Azure RBAC(역할 기반 액세스 제어) 역할을 할당해야 합니다. 특히 세션 호스트에 로그인하려면 *가상 머신 사용자 로그인* 역할이 필요하며 로컬 관리자 권한에는 *가상 머신 관리자 로그인* 역할이 필요합니다. 

1. 랩 컴퓨터의 Azure Portal을 표시하는 웹 브라우저에서 **리소스 그룹**을 검색하여 선택하고 **리소스 그룹** 페이지에서 **az140-21e-RG**를 선택합니다.
1. **az140-21e-RG** 페이지의 세로 탐색 메뉴에서 **액세스 제어(IAM)** 를 선택합니다.
1. **az140-21e-RG\|액세스 제어(IAM)** 페이지에서 **+ 추가**를 선택하고 드롭다운 메뉴에서 **역할 할당 추가**를 선택합니다.
1. **역할 할당 추가** 페이지의 **역할** 탭에서 **작업 함수 역할** 탭이 선택되어 있는지 확인하고, 검색 텍스트 상자에 **가상 머신 사용자 로그인**을 입력하고, 결과 목록에서 **가상 머신 사용자 로그인**을 선택한 다음, **다음**을 선택합니다.
1. **역할 할당 추가** 페이지의 **구성원** 탭에서 **사용자, 그룹 또는 서비스 주체** 옵션이 선택되어 있는지 확인하고 , **+ 구성원 선택**을 클릭하고, **구성원 선택** 창에서 이 연습의 첫 번째 작업에서 식별한 **AVD-RemoteApp** 그룹을 찾은 다음 **선택**을 클릭합니다.
1. **역할 할당 추가** 페이지의 **구성원** 탭으로 돌아가서 **다음**을 선택합니다.
1. **역할 할당 추가** 페이지의 **할당 유형** 탭에서 **할당 유형**을 **활성**으로 설정한 다음 **검토 + 할당**을 선택합니다.
1. **역할 할당 추가** 페이지의 **검토 + 할당** 탭에서 **검토 + 할당**을 선택합니다. 
1. **az140-21e-RG\|엑세스 제어(IAM)** 페이지로 돌아가서 **+ 추가**를 선택하고 드롭다운 메뉴에서 **역할 할당 추가**를 선택합니다.
1. **역할 할당 추가** 페이지의 **역할** 탭에서 **작업 함수 역할** 탭이 선택되어 있는지 확인하고 검색 텍스트 상자에 **Virtual Machine Administrator Login**을 입력하고 결과 목록에서 **가상 머신 관리자 로그인**을 선택한 다음 **다음**을 선택합니다.
1. **역할 할당 추가** 페이지의 **구성원** 탭에서 **사용자, 그룹 또는 서비스 주체** 옵션이 선택되어 있는지 확인하고, **+ 구성원 선택**을 클릭하고, **구성원 선택** 창에서 이 연습의 첫 번째 작업에서 식별한 **AVD-DAG** 그룹을 찾은 다음 **선택**을 클릭합니다.
1. **역할 할당 추가** 페이지의 **구성원** 탭으로 돌아가 **할당 유형**을 **활성**으로 설정하고 **검토 + 할당**을 선택한 다음 **검토 + 할당**을 다시 선택합니다. 
