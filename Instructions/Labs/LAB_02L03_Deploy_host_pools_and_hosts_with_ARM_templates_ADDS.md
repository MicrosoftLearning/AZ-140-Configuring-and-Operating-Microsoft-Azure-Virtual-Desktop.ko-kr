---
lab:
  title: '랩: AD DS(Azure Resource Manager 템플릿)를 사용하여 호스트 풀 및 호스트 배포'
  module: 'Module 2: Implement an AVD Infrastructure'
---

# 랩 - Azure Resource Manager 템플릿을 사용하여 호스트 풀 및 호스트 배포
# 학생용 랩 매뉴얼

## 랩 종속성

- 이 랩에서 사용할 Azure 구독입니다.
- 이 랩에서 사용할 Azure 구독의 소유자 또는 기여자 역할이 있는 Microsoft 계정 또는 Microsoft Entra 계정 및 해당 Azure 구독과 연결된 Microsoft Entra 테넌트의 Global 관리istrator 역할.
- 완료된 **Azure Virtual Desktop의 배포 준비(AD DS)** 랩
- 완료된 랩 **은 AD DS(Azure Portal)를 사용하여 호스트 풀 및 세션 호스트를 배포합니다.**

## 예상 소요 시간

45분

## 실습 시나리오

Azure Resource Manager 템플릿을 사용하여 Azure Virtual Desktop 호스트 풀 및 호스트의 배포를 자동화해야 합니다.

## 목표
  
이 랩을 완료하면 다음을 수행할 수 있습니다.

- Azure Resource Manager 템플릿을 사용하여 Azure Virtual Desktop 호스트 풀 및 호스트 배포

## 랩 파일

-  \\\\AZ-140\\AllFiles\\Labs\\02\\az140-23_azuredeployhp23.parameters.json
-  \\\\AZ-140\\AllFiles\\Labs\\02\\az140-23_azuremodifyhp23.parameters.json

## 지침

### 연습 1: Azure Resource Manager 템플릿을 사용하여 Azure Virtual Desktop 호스트 풀 및 호스트 배포
  
이 연습의 주요 작업은 다음과 같습니다.

1. Azure Resource Manager 템플릿을 사용하여 Azure Virtual Desktop 호스트 풀 배포 준비
1. Azure Resource Manager 템플릿을 사용하여 Azure Virtual Desktop 호스트 풀 및 호스트 배포
1. Azure Virtual Desktop 호스트 풀 및 호스트 배포 확인
1. Azure Resource Manager 템플릿을 사용하여 기존 Azure Virtual Desktop 호스트 풀에 호스트 추가 준비
1. Azure Resource Manager 템플릿을 사용하여 기존 Azure Virtual Desktop 호스트 풀에 호스트 추가
1. Azure Virtual Desktop 호스트 풀에 대한 변경 내용 확인
1. Azure Virtual Desktop 호스트 풀에서 개인 데스크톱 할당 관리

#### 작업 1: Azure Resource Manager 템플릿을 사용하여 Azure Virtual Desktop 호스트 풀 배포 준비

1. 랩 컴퓨터에서 웹 브라우저를 시작하고, Azure Portal[로 ](https://portal.azure.com)이동하고, 이 랩에서 사용할 구독에서 소유자 역할이 있는 사용자 계정의 자격 증명을 제공하여 로그인합니다.
1. Azure Portal에서 가상 머신을 검색하여 선택하고 **가상 머신**** 블레이드에서 **az140-dc-vm11**을 선택합니다**.
1. **az140-dc-vm11** 블레이드에서 **연결**을 선택하고 드롭다운 메뉴에서 **베스천**을 선택합니다. **az140-dc-vm11 \| 연결** 블레이드의 **베스천** 탭에서 **베스천 사용**을 선택합니다.
1. 메시지가 표시되면 다음 자격 증명을 제공하고 **연결**을 선택합니다.

   |설정|값|
   |---|---|
   |사용자 이름|**Student**|
   |암호|**Pa55w.rd1234**|

1. az140-dc-vm11에 대한 **Bastion 세션 내에서 Windows PowerShell ISE**를 관리자로 시작**** 합니다.
1. az140-dc-vm11에 대한 **Bastion 세션 내에서 관리istrator: Windows PowerShell ISE** 콘솔에서 **다음을 실행하여 Azure Virtual Desktop 풀 호스트의 컴퓨터 개체를 호스팅할 WVDInfra**라는 **조직 단위의 고유 이름을 식별**합니다.

   ```powershell
   (Get-ADOrganizationalUnit -Filter "Name -eq 'WVDInfra'").distinguishedName
   ```

1. az140-dc-vm11에 대한 **Bastion 세션 내에서 관리istrator: Windows PowerShell ISE** 스크립트 창에서 **다음을 실행하여 Azure Virtual Desktop 호스트를 AD DS에 조인하는 데 사용할 ADATUM\\Student** 계정의 **사용자 계정 이름 특성을 식별합니다기본(**student@adatum.com**).**

   ```powershell
   (Get-ADUser -Filter "sAMAccountName -eq 'student'").userPrincipalName
   ```

1. az140-dc-vm11에 대한 **Bastion 세션 내에서 관리istrator: Windows PowerShell ISE** 스크립트 창에서 다음을 실행하여 이 랩의 뒷부분에서 **개인 데스크톱 할당을 테스트하는 데 사용할 ADATUM aduser7** 및 **ADATUM\\\\aduser8** 계정의 **사용자 계정 이름을 식별**합니다.

   ```powershell
   (Get-ADUser -Filter "sAMAccountName -eq 'aduser7'").userPrincipalName
   (Get-ADUser -Filter "sAMAccountName -eq 'aduser8'").userPrincipalName
   ```

   > **참고**: 식별한 모든 사용자 계정 이름 값을 기록합니다. 이 랩의 뒷부분에서 계정 이름이 필요합니다.

1. az140-dc-vm11에 대한 **Bastion 세션 내에서 관리istrator: Windows PowerShell ISE** 스크립트 창에서 **다음을 실행하여 템플릿 기반 배포를 수행하는 데 필요한 토큰 만료 시간을 계산**합니다.

   ```powershell
   $((get-date).ToUniversalTime().AddDays(1).ToString('yyyy-MM-ddTHH:mm:ss.fffffffZ'))
   ```

   > **참고**: 값은 형식 `2022-03-27T00:51:28.3008055Z`과 유사해야 합니다. 다음 작업에서 필요하므로 기록합니다.

   > **참고**: 호스트가 풀에 조인할 수 있도록 권한을 부여하려면 등록 토큰이 필요합니다. 토큰의 만료 날짜 값은 현재 날짜와 시간으로부터 1시간에서 1개월 사이여야 합니다.

1. az140-dc-vm11에 대한 **Bastion 세션 내에서 Microsoft Edge를 시작하고 Azure Portal[로 이동합니다](https://portal.azure.com).** 메시지가 표시되면 이 랩에서 사용 중인 구독에서 소유자 역할이 있는 사용자 계정의 Microsoft Entra 자격 증명을 사용하여 로그인합니다.
1. az140-dc-vm11에 대한 **Bastion 세션 내에서 Azure Portal에서 Azure Portal 페이지의 맨 위에 있는 검색 리소스, 서비스 및 문서** 텍스트 상자를 사용하여 **가상 네트워크를 검색 및 탐색**하고 가상 네트워크**** 블레이드에서 **az140-adds-vnet11을** 선택합니다**.** 
1. az140-adds-vnet11** 블레이드에서 서브넷을 선택하고 **, 서브넷** 블레이드에서 **+ 서브넷**을 선택하고 **, 서브넷**** 추가 블레이드에서 **다음 설정을 지정하고(다른 모든 설정을 기본값으로 유지) 저장**을 클릭합니다**.**

   |설정|값|
   |---|---|
   |속성|**hp2-Subnet**|
   |서브넷 주소 범위|**10.0.2.0/24**|

1. az140-dc-vm11에 대한 **Bastion 세션 내에서 Azure Portal에서 Azure Portal 페이지의 맨 위에 있는 검색 리소스, 서비스 및 문서** 텍스트 상자를 사용하여 **네트워크 보안 그룹을 검색 및 탐색**하고 네트워크 **보안 그룹**** 블레이드에서 **az140-11-RG** 리소스 그룹에서 네트워크 보안 그룹을** 선택합니다.
1. 네트워크 보안 그룹 블레이드의 왼쪽 **세로 메뉴에서 설정** 섹션에서 속성을** 클릭합니다**.
1. 속성** 블레이드의 **리소스 ID** 텍스트 상자 오른쪽에 있는 클립보드**로 복사 아이콘을 **클릭합니다**. 

   > **참고**: 구독 ID는 다르지만 값은 형식 `/subscriptions/de8279a3-0675-40e6-91e2-5c3728792cb5/resourceGroups/az140-11-RG/providers/Microsoft.Network/networkSecurityGroups/az140-cl-vm11-nsg`과 유사해야 합니다. 다음 작업에서 필요하므로 기록합니다.

#### 작업 2: Azure Resource Manager 템플릿을 사용하여 Azure Virtual Desktop 호스트 풀 및 호스트 배포

1. 랩 컴퓨터에서 웹 브라우저를 시작하고, Azure Portal[로 ](https://portal.azure.com)이동하고, 이 랩에서 사용할 구독에서 소유자 역할이 있는 사용자 계정의 자격 증명을 제공하여 로그인합니다.
1. 랩 컴퓨터의 동일한 웹 브라우저 창에서 다른 웹 브라우저 탭을 열고 GitHub Azure RDS 템플릿 리포지토리 페이지 [ARM 템플릿으로 이동하여 새 Azure Virtual Desktop 호스트 풀](https://github.com/Azure/RDS-Templates/tree/master/ARM-wvd-templates/CreateAndProvisionHostPool)을 만들고 프로비전합니다. 
1. ARM 템플릿에서 **새 Azure Virtual Desktop 호스트 풀**을 만들고 프로비전하려면 Azure**에 배포를 선택합니다**. 그러면 브라우저가 Azure Portal의 **사용자 지정 배포** 블레이드로 자동으로 리디렉션됩니다.
1. 사용자 지정 배포 블레이드에서 **매개 변수** 편집을 선택합니다**.**
1. **매개 변수 편집** 블레이드의 **열기** 대화 상자에서 **파일 로드**를 선택하고 **\\\\AZ-140\\AllFiles\\Labs\\02\\az140-23_azuredeployhp23.parameters.json**을 선택한 후에 **열기**, **저장**을 차례로 선택합니다. 
1. 사용자 지정 배포** 블레이드로 **돌아가서 다음 설정을 지정합니다(다른 설정은 기존 값으로 유지).

   |설정|값|
   |---|---|
   |구독|이 랩에서 사용 중인 Azure 구독의 이름|
   |리소스 그룹|새 리소스 그룹 **az140-23-RG**의 이름|
   |지역|**Azure Virtual Desktop의 배포 준비(AD DS)** 랩에서 AD DS 도메인 컨트롤러를 호스트하는 Azure VM을 배포했던 Azure 지역의 이름|
   |위치|지역 매개 변수의 값 **으로 설정된 것과 동일한 Azure 지역의** 이름입니다.|
   |작업 영역 위치|지역 매개 변수의 값 **으로 설정된 것과 동일한 Azure 지역의** 이름입니다.|
   |작업 영역 리소스 그룹|none, null이면 해당 값이 배포 대상 리소스 그룹과 일치하도록 자동으로 설정되기 때문에|
   |모든 애플리케이션 그룹 참조|없음, 대상 작업 영역에 기존 애플리케이션 그룹이 없으므로(작업 영역 없음)|
   |Vm 위치|위치** 매개 변수의 값**으로 설정된 것과 동일한 Azure 지역의 이름입니다.|
   |네트워크 보안 그룹 만들기|**false**|
   |네트워크 보안 그룹 ID|이전 작업에서 식별한 기존 네트워크 보안 그룹의 resourceID 매개 변수 값|
   |토큰 만료 시간| 이전 작업에서 계산한 토큰 만료 시간의 값입니다.|

   > **참고**: 배포는 개인 데스크톱 할당 유형으로 풀을 프로비전합니다.

1. 사용자 지정 배포 블레이드에서 **검토 + 만들기**를 선택하고 **만들기**를 선택합니다**.**

   > **참고**: 배포가 완료될 때까지 기다린 후 다음 작업을 진행하세요. 15분 정도 걸릴 수 있습니다. 

#### 작업 3: Azure Virtual Desktop 호스트 풀 및 호스트 배포 확인

1. 랩 컴퓨터에서 Azure Portal이 표시된 웹 브라우저에서 **Azure Virtual Desktop**을 검색하여 선택하고 **Azure Virtual Desktop** 블레이드에서 **호스트 풀**을 선택하고 **Azure Virtual Desktop \| 호스트 풀** 블레이드에서 새로 배포된 풀을 나타내는 **az140-23-hp2** 항목을 선택합니다.
1. **az140-23-hp2** 블레이드의 왼쪽에 있는 세로 **메뉴의 관리** 섹션에서 세션 호스트를** 클릭합니다**. 
1. **az140-23-hp2 \| 세션 호스트** 블레이드에서 배포가 두 개의 호스트로 구성되어 있는지 확인합니다.
1. **az140-23-hp2 \| 세션 호스트** 블레이드 왼쪽의 세로 메뉴에 있는 **관리** 섹션에서 **애플리케이션 그룹**을 클릭합니다.
1. **az140-23-hp2 \| 애플리케이션 그룹** 블레이드에서 배포에 **Default Desktop** 애플리케이션 그룹 **az140-23-hp2-DAG**가 포함되어 있는지 확인합니다.

#### 작업 4: Azure Resource Manager 템플릿을 사용하여 기존 Azure Virtual Desktop 호스트 풀에 호스트 추가 준비

1. 랩 컴퓨터에서 Bastion 세션으로 전환하여 **az140-dc-vm11**로 전환합니다. 
1. az140-dc-vm11에 대한 **Bastion 세션 내에서 관리istrator: Windows PowerShell ISE** 콘솔에서 다음을 실행하여 이 연습의 앞부분에서 **프로비전한 풀에 새 호스트를 조인하는 데 필요한 토큰을 생성**합니다.

   ```powershell
   $registrationInfo = New-AzWvdRegistrationInfo -ResourceGroupName 'az140-23-RG' -HostPoolName 'az140-23-hp2' -ExpirationTime $((get-date).ToUniversalTime().AddDays(1).ToString('yyyy-MM-ddTHH:mm:ss.fffffffZ'))
   ```

1. az140-dc-vm11**에 대한 **Bastion 세션 내에서 관리istrator: Windows PowerShell ISE** 콘솔에서 **다음을 실행하여 토큰 값을 검색하고 클립보드에 붙여넣습니다.

   ```powershell
   $registrationInfo.Token | clip
   ```

   > **참고**: 클립보드에 복사한 값을 기록합니다(예: 메모장 시작하고 Ctrl+V 키 조합을 눌러 클립보드의 콘텐츠를 메모장 붙여넣음) 다음 작업에서 필요하므로 해당 내용을 기록합니다. 사용 중인 값에 줄 바꿈 없이 한 줄의 텍스트가 포함되어 있는지 확인합니다. 

   > **참고**: 호스트가 풀에 조인할 수 있도록 권한을 부여하려면 등록 토큰이 필요합니다. 토큰의 만료 날짜 값은 현재 날짜와 시간으로부터 1시간에서 1개월 사이여야 합니다.

#### 작업 5: Azure Resource Manager 템플릿을 사용하여 기존 Azure Virtual Desktop 호스트 풀에 호스트 추가

1. 랩 컴퓨터의 동일한 웹 브라우저 창에서 다른 웹 브라우저 탭을 열고 GitHub Azure RDS 템플릿 리포지토리 페이지 [ARM 템플릿으로 이동하여 기존 Azure Virtual Desktop 호스트 풀](https://github.com/Azure/RDS-Templates/tree/master/ARM-wvd-templates/AddVirtualMachinesToHostPool)에 sessionhosts를 추가합니다. 
1. ARM 템플릿에서 **기존 Azure Virtual Desktop 호스트 풀**에 sessionhosts를 추가하려면 Azure**에 배포를 선택합니다**. 그러면 브라우저가 Azure Portal의 **사용자 지정 배포** 블레이드로 자동으로 리디렉션됩니다.
1. 사용자 지정 배포 블레이드에서 **매개 변수** 편집을 선택합니다**.**
1. **매개 변수 편집** 블레이드의 **열기** 대화 상자에서 **파일 로드**를 선택하고 **\\\\AZ-140\\AllFiles\\Labs\\02\\az140-23_azuremodifyhp23.parameters.json**을 선택한 후에 **열기**, **저장**을 차례로 선택합니다. 
1. 사용자 지정 배포** 블레이드로 **돌아가서 다음 설정을 지정합니다(다른 설정은 기존 값으로 유지).

   |설정|값|
   |---|---|
   |구독|이 랩에서 사용 중인 Azure 구독의 이름|
   |리소스 그룹|**az140-23-RG**|
   |Hostpool 토큰|이전 작업에서 생성한 토큰의 값|
   |호스트 풀 위치|이 랩의 앞부분에서 hostpool을 배포한 Azure 지역의 이름|
   |Vm 관리istrator 계정 사용자 이름|**학생** @adatum.com 사용 안 함|
   |Vm 관리istrator 계정 암호|**Pa55w.rd1234**|
   |Vm 위치|Hostpool 위치** 매개 변수의 값**으로 설정된 것과 동일한 Azure 지역의 이름|
   |네트워크 보안 그룹 만들기|**false**|
   |네트워크 보안 그룹 ID|이전 작업에서 식별한 기존 네트워크 보안 그룹의 resourceID 매개 변수 값|

1. 사용자 지정 배포 블레이드에서 **검토 + 만들기**를 선택하고 **만들기**를 선택합니다**.**

   > **참고**: 배포가 완료될 때까지 기다린 후 다음 작업을 진행하세요. 5분 정도 걸릴 수 있습니다.

#### 작업 6: Azure Virtual Desktop 호스트 풀에 대한 변경 내용 확인

1. 랩 컴퓨터의 Azure Portal을 표시하는 웹 브라우저에서 가상 머신을 검색하여 선택하고 **가상 머신**** 블레이드에서 **목록에 az140-23-p2-2**라는 **추가 가상 머신이 포함되어 있습니다.
1. 랩 컴퓨터에서 Bastion 세션으로 전환하여 **az140-dc-vm11**로 전환합니다. 
1. az140-dc-vm11**에 대한 **Bastion 세션 내에서 관리istrator: Windows PowerShell ISE** 콘솔에서 **다음을 실행하여 세 번째 호스트가 adatum.com** AD DS에 성공적으로 조인되었는지 **확인합니다기본

   ```powershell
   Get-ADComputer -Filter "sAMAccountName -eq 'az140-23-p2-2$'"
   ```
1. 랩 컴퓨터로 다시 전환한 후 Azure Portal이 표시된 웹 브라우저에서 **Azure Virtual Desktop**을 검색하여 선택하고 **Azure Virtual Desktop** 블레이드에서 **호스트 풀**을 선택하고 **Azure Virtual Desktop \| 호스트 풀** 블레이드에서 새로 수정된 풀을 나타내는 **az140-23-hp2** 항목을 선택합니다.
1. **az140-23-hp2** 블레이드에서 Essentials 섹션을** 검토하고 **할당 유형**이 **자동**으로 **설정된 호스트 풀 유형**이 개인**** 으로 **설정되어 있는지 확인합니다.
1. **az140-23-hp2** 블레이드의 왼쪽에 있는 세로 **메뉴의 관리** 섹션에서 세션 호스트를** 클릭합니다**. 
1. **az140-23-hp2 \| 세션 호스트** 블레이드에서 배포가 세 개의 호스트로 구성되어 있는지 확인합니다. 

#### 작업 7: Azure Virtual Desktop 호스트 풀에서 개인 데스크톱 할당 관리

1. 랩 컴퓨터에서 Azure Portal을 표시하는 웹 브라우저의 **az140-23-hp2 \| 세션 호스트** 블레이드의 왼쪽 세로 메뉴에 있는 **관리** 섹션의 **애플리케이션 그룹**을 선택합니다. 
1. **az140-23-hp2 \| 애플리케이션 그룹** 블레이드의 애플리케이션 그룹 목록에서 **az140-23-hp2-DAG**를 선택합니다.
1. **az140-23-hp2-DAG** 블레이드의 왼쪽 세로 메뉴에서 할당을** 선택합니다**. 
1. **az140-23-hp2-DAG \| 할당** 블레이드에서 **+ 추가**를 선택합니다.
1. **Microsoft Entra 사용자 또는 사용자 그룹** 선택 블레이드에서 az140-wvd-personal**을 선택하고 **선택을** 클릭합니다**.

   > **참고**: 이제 Azure Virtual Desktop 호스트 풀에 연결하는 사용자의 환경을 검토해 보겠습니다.

1. 랩 컴퓨터의 Azure Portal을 표시하는 브라우저 창에서 가상 머신을 검색하여 선택하고 **가상 머신**** 블레이드에서 **az140-cl-vm11** 항목을 선택합니다**.
1. **az140-cl-vm11** 블레이드에서 **연결**을 선택하고 드롭다운 메뉴에서 **베스천**을 선택합니다. **az140-cl-vm11 \| 연결** 블레이드의 **베스천** 탭에서 **베스천 사용**을 선택합니다.
1. 메시지가 표시되면 다음 자격 증명을 제공하고 **연결**을 선택합니다.

   |설정|값|
   |---|---|
   |사용자 이름|**Student@adatum.com**|
   |암호|**Pa55w.rd1234**|

1. az140-cl-vm11에 대한 **Bastion 세션 내에서 시작을** **클릭하고 **시작** 메뉴에서 원격 데스크톱** 클라이언트 앱을 선택합니다**.**
2. 원격 데스크톱 창에서 오른쪽 위 모서리에 있는 줄임표 아이콘을 클릭하고 드롭다운 메뉴에서 **구독 취소**를 클릭한 다음, 확인 메시지가 표시되면 **계속**을 클릭합니다.
3. az140-cl-vm11에 대한 **Bastion 세션 내에서 원격 데스크톱 창**의 시작 하자 페이지에서** 구독**을 클릭합니다**.**
4. **원격 데스크톱** 클라이언트 창에서 구독**을 선택하고 **메시지가 표시되면 해당 userPrincipalName 및 이 사용자 계정을 만들 때 설정한 암호를 제공하여 aduser7** 자격 증명으로 로그인**합니다.

   > **참고**: 또는 원격 데스크톱** 클라이언트 창에서 **URL**로 구독을 선택하고**, 작업 영역 구독 창의 **전자 메일 또는 작업 영역** URL**에서 **다음을 입력****https://rdweb.wvd.microsoft.com/api/arm/feeddiscovery하고, 다음**을 선택하고**, 메시지가 표시되면 aduser7** 자격 증명으로 로그인**합니다(사용자 이름 및 이 계정을 만들 때 설정한 암호로 userPrincipalName 특성 사용). 

1. **원격 데스크톱** 페이지에서 **SessionDesktop** 아이콘을 두 번 클릭합니다. 자격 증명을 입력하라는 메시지가 표시되면 동일한 암호를 다시 입력하고 **Remember me** 체크박스를 선택한 후에 **확인**을 클릭합니다.
1. **모든 앱**에 로그인된 상태로 유지 창에서 검사함의 선택을 취소하고 **조직에서 내 디바이스**를 관리할 수 있도록 허용 검사box를 선택하고 아니요를 선택하고 **이 앱에만** 로그인합니다. 
1. aduser7**이 **원격 데스크톱을 통해 호스트에 성공적으로 로그인했는지 확인합니다.
1. aduser7**로 호스트 중 하나에 대한 원격 데스크톱 세션 내에서 시작을** 마우스 오른쪽 단추로 **클릭하고**, 마우스 오른쪽 단추로 클릭 메뉴에서 종료 또는 로그아웃**을 선택하고**, 계단식 메뉴에서 로그아웃**을 클릭합니다**.

   > **참고**: 이제 개인 데스크톱 할당을 직접 모드에서 자동 모드로 전환해 보겠습니다. 

1. 랩 컴퓨터로 전환하여 Azure Portal 표시하는 웹 브라우저로 전환하고 **az140-23-hp2-DAG \| 할당** 블레이드의 할당 목록 바로 위에 있는 정보 표시줄에서 **VM 할당** 링크를 클릭합니다. 그러면 **az140-23-hp2 \| 세션 호스트** 블레이드로 리디렉션됩니다. 
1. **az140-23-hp2 \| 세션 호스트** 블레이드에서 호스트 중 하나의 **할당된 사용자** 열 목록에 **aduser7**이 표시되어 있는지 확인합니다.

   > **참고**: 호스트 풀이 자동 할당에 대해 구성되었으므로 이 작업이 필요합니다.

1. 랩 컴퓨터의 Azure Portal을 표시하는 웹 브라우저 창에서 Cloud Shell 창 내에서 **PowerShell** 셸** 세션을 엽니다**.
1. Cloud Shell 창의 PowerShell 세션에서 다음을 실행하여 직접 할당 모드로 전환합니다.

    ```powershell
    Update-AzWvdHostPool -ResourceGroupName 'az140-23-RG' -Name 'az140-23-hp2' -PersonalDesktopAssignmentType Direct
    ```

1. 랩 컴퓨터의 Azure Portal을 표시하는 웹 브라우저 창에서 az140-23-hp2** 호스트 풀 블레이드로 이동하여 **Essentials 섹션을** 검토하고 **할당 유형**이 Direct**로 **설정된 호스트 풀 유형**이 **개인**** 으로 설정되어 **있는지 확인합니다.
1. Bastion 세션**으로 다시 az140-cl-vm11**로 전환하고, 원격 데스크톱** 창에서 **오른쪽 위 모서리에 있는 줄임표 아이콘을 클릭하고, 드롭다운 메뉴에서 구독** 취소를 클릭하고**, 확인 메시지가 표시되면 [계속 **]을 클릭합니다**.
1. az140-cl-vm11에 대한 **Bastion 세션 내에서 원격 데스크톱** 창**의 시작** 하자 페이지에서 구독**을 클릭합니다**.****
1. 로그인하라는 메시지가 표시되면 **계정 선택** 창에서 **다른 계정 사용**을 클릭하고 메시지가 표시되면 **aduser8** 사용자 계정의 사용자 계정 이름을 사용하여 로그인합니다. 암호는 이 계정을 만들 때 설정한 암호입니다.
1. **모든 앱**에 로그인된 상태로 유지 창에서 검사함의 선택을 취소하고 **조직에서 내 디바이스**를 관리할 수 있도록 허용 검사box를 선택하고 아니요를 선택하고 **이 앱에만** 로그인합니다. 
1. **원격 데스크톱** 페이지에서 SessionDesktop** 아이콘을 두 번 클릭하고 **현재 사용 가능한 리소스가 없으므로 연결할 수 없다는 오류 메시지가 **표시되는지 확인합니다. 나중에 다시 시도하거나 기술 지원팀에 문의하여 계속 도움을** 받으려면 [확인 **]을 클릭하십시오**.

   > **참고**: 이는 호스트 풀이 직접 할당에 대해 구성되고 **aduser8** 이 호스트에 할당되지 않았기 때문에 예상됩니다.

1. 랩 컴퓨터에서 Azure Portal을 표시하는 웹 브라우저로 전환하고 **az140-23-hp2 \| 세션 호스트** 블레이드에서 할당되지 않은 나머지 호스트 2개 중 하나 옆에 있는 **할당된 사용자** 열에서 **(할당)** 링크를 선택합니다.
1. 사용자 할당에서 **aduser8**을 선택하고 선택을 **** 클릭한 **다음 확인 메시지가 표시되면 [확인 **]을 클릭합니다**.**
1. Bastion 세션**으로 다시 az140-cl-vm11**로 전환하고, 원격 데스크톱** 창에서 **SessionDesktop** 아이콘을 두 번 클릭합니다**. 암호를 묻는 메시지가 표시되면 이 사용자 계정을 만들 때 설정한 암호를 입력하고 확인을 클릭하고 **** 할당된 호스트에 성공적으로 로그인할 수 있는지 확인합니다.

### 연습 2: 랩에서 프로비전된 Azure VM 중지 및 할당 취소

이 연습의 주요 작업은 다음과 같습니다.

1. 랩에서 프로비전된 Azure VM 중지 및 할당 취소

>**참고**: 이 연습에서는 이 랩에 프로비전된 Azure VM의 할당을 취소하여 해당 컴퓨팅 요금을 최소화합니다.

#### 작업 1: 랩에서 프로비전된 Azure VM 할당 취소

1. 랩 컴퓨터로 전환하고 Azure Portal을 표시하는 웹 브라우저 창에서 Cloud Shell 창 내에서 **PowerShell** 셸** 세션을 엽니다**.
1. Cloud Shell 창의 PowerShell 세션에서 다음을 실행하여 이 랩에서 만든 모든 Azure VM을 나열합니다.

   ```powershell
   Get-AzVM -ResourceGroup 'az140-23-RG'
   ```

1. Cloud Shell 창의 PowerShell 세션에서 다음을 실행하여 이 랩에서 만든 모든 Azure VM을 중지하고 할당을 취소합니다.

   ```powershell
   Get-AzVM -ResourceGroup 'az140-23-RG' | Stop-AzVM -NoWait -Force
   ```

   >**참고**: 명령은 -NoWait 매개 변수에 의해 결정된 대로 비동기적으로 실행되므로 동일한 PowerShell 세션 내에서 즉시 다른 PowerShell 명령을 실행할 수 있지만 Azure VM이 실제로 중지되고 할당이 취소되기까지 몇 분 정도 걸립니다.
