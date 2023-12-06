---
lab:
  title: '랩: 세션 호스트 이미지 만들기 및 관리(AD DS)'
  module: 'Module 2: Implement an AVD Infrastructure'
---

# 랩 - AD DS(세션 호스트 이미지) 만들기 및 관리
# 학생용 랩 매뉴얼

## 랩 종속성

- 이 랩에서 사용할 Azure 구독입니다.
- 이 랩에서 사용할 Azure 구독의 소유자 또는 기여자 역할이 있는 Microsoft 계정 또는 Microsoft Entra 계정 및 해당 Azure 구독과 연결된 Microsoft Entra 테넌트의 Global 관리istrator 역할.
- 완료된 **Azure Virtual Desktop의 배포 준비(AD DS)** 랩

## 예상 소요 시간

60분

## 랩 시나리오

Microsoft Entra DS 환경에서 Azure Virtual Desktop 호스트 이미지를 만들고 관리해야 합니다.

## 목표
  
이 랩을 완료하면 다음을 수행할 수 있습니다.

- WVD 세션 호스트 이미지 만들기 및 관리

## 랩 파일

-  \\\\AZ-140\\AllFiles\\Labs\\02\\az140-25_azuredeployvm25.json
-  \\\\AZ-140\\AllFiles\\Labs\\02\\az140-25_azuredeployvm25.parameters.json

## 지침

### 연습 1: 세션 호스트 이미지 만들기 및 관리
  
이 연습의 주요 작업은 다음과 같습니다.

1. Azure Virtual Desktop 호스트 이미지 구성 준비
1. Azure Bastion 배포
1. Azure Virtual Desktop 호스트 이미지 구성
1. Azure Virtual Desktop 호스트 이미지 만들기
1. 사용자 지정 이미지를 사용하여 Azure Virtual Desktop 호스트 풀 프로비전

#### 작업 1: Azure Virtual Desktop 호스트 이미지 구성 준비

1. 랩 컴퓨터에서 웹 브라우저를 시작하고, Azure Portal[로 ](https://portal.azure.com)이동하고, 이 랩에서 사용할 구독에서 소유자 역할이 있는 사용자 계정의 자격 증명을 제공하여 로그인합니다.
1. Azure Portal에서 검색 텍스트 상자 오른쪽에 있는 도구 모음 아이콘을 선택하여 Cloud Shell** 창을 엽니다**.
1. **Bash**와 **PowerShell** 중에서 선택하라는 메시지가 표시되면 **PowerShell**을 선택합니다. 
1. 랩 컴퓨터의 Azure Portal을 표시하는 웹 브라우저의 Cloud Shell 창에 있는 PowerShell 세션에서 다음을 실행하여 Azure Virtual Desktop 호스트 이미지를 포함하는 리소스 그룹을 만듭니다.

   ```powershell
   $vnetResourceGroupName = 'az140-11-RG'
   $location = (Get-AzResourceGroup -ResourceGroupName $vnetResourceGroupName).Location
   $imageResourceGroupName = 'az140-25-RG'
   New-AzResourceGroup -Location $location -Name $imageResourceGroupName
   ```

1. Azure Portal의 Cloud Shell 창 도구 모음에서 **파일 업로드/다운로드** 아이콘을 선택하고 드롭다운 메뉴에서 **업로드**를 선택합니다. 그런 다음, **\\\\AZ-140\\AllFiles\\Labs\\02\\az140-25_azuredeployvm25.json** 및 **\\\\AZ-140\\AllFiles\\Labs\\02\\az140-25_azuredeployvm25.parameters.json** 파일을 Cloud Shell 홈 디렉터리에 업로드합니다.
1. Cloud Shell 창의 PowerShell 세션에서 다음을 실행하여 새로 만든 서브넷에 Azure Virtual Desktop 클라이언트 역할을 하는 Windows 10을 실행하는 Azure VM을 배포합니다.

   ```powershell
   New-AzResourceGroupDeployment `
     -ResourceGroupName $imageResourceGroupName `
     -Name az140lab0205vmDeployment `
     -TemplateFile $HOME/az140-25_azuredeployvm25.json `
     -TemplateParameterFile $HOME/az140-25_azuredeployvm25.parameters.json
   ```

   > **참고**: 배포가 완료될 때까지 기다린 후 다음 연습을 진행하세요. 배포는 10분 정도 걸릴 수 있습니다.

#### 작업 2: Azure Bastion 배포 

> **참고**: Azure Bastion은 이 연습의 이전 작업에서 배포한 퍼블릭 엔드포인트 없이 Azure VM에 연결할 수 있으며 운영 체제 수준 자격 증명을 무차별 악용하지 못하도록 보호합니다.

> **참고**: 브라우저에 팝업 기능이 사용하도록 설정되어 있는지 확인합니다.

1. Azure Portal을 표시하는 브라우저 창에서 다른 탭을 열고 브라우저 탭에서 Azure Portal로 이동합니다.
1. Azure Portal에서 검색 텍스트 상자 오른쪽에 있는 도구 모음 아이콘을 선택하여 Cloud Shell** 창을 엽니다**.
1. Cloud Shell 창의 PowerShell 세션에서 다음을 실행하여 **AzureBastionSubnet**이라는 서브넷을 이 연습의 앞부분에서 만든 가상 네트워크 **az140-25-vnet**에 추가합니다.

   ```powershell
   $resourceGroupName = 'az140-25-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'az140-25-vnet'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
     -Name 'AzureBastionSubnet' `
     -AddressPrefix 10.25.254.0/24 `
     -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. Cloud Shell 창을 닫습니다.
1. Azure Portal에서 **베스천**을 검색하여 선택하고 **베스천** 블레이드에서 **+ 만들기**를 선택합니다.
1. **베스천 만들기** 블레이드의 **기본 사항** 탭에서 다음 설정을 지정하고 **검토 + 만들기**를 선택합니다.

   |설정|값|
   |---|---|
   |구독|이 랩에서 사용 중인 Azure 구독의 이름|
   |Resource group|**az140-25-RG**|
   |이름|**az140-25-bastion**|
   |지역|이 연습의 이전 작업에서 리소스를 배포한 동일한 Azure 지역|
   |계층|**기본**|
   |가상 네트워크|**az140-25-vnet**|
   |서브넷|**AzureBastionSubnet(10.25.254.0/24)**|
   |공용 IP 주소|**새로 만들기**|
   |공용 IP 이름|**az140-25-vnet-ip**|

1. **베스천 만들기** 블레이드의 **검토 + 만들기** 탭에서 **만들기**를 선택합니다.

   > **참고**: 배포가 완료될 때까지 기다린 후 다음 연습을 진행하세요. 배포는 5분 정도 걸릴 수 있습니다.

#### 작업 3: Azure Virtual Desktop 호스트 이미지 구성

1. Azure Portal에서 가상 머신을 검색하여 선택하고 **가상 머신**** 블레이드에서 **az140-25-vm0**을 선택합니다**.
1. **az140-25-vm0** 블레이드에서 **연결**을 선택하고 드롭다운 메뉴에서 **베스천**을 선택합니다. **az140-25-vm0 \| 연결** 블레이드의 **베스천** 탭에서 **베스천 사용**을 선택합니다.
1. 메시지가 표시되면 다음 자격 증명을 제공하고 **연결**을 선택합니다.

   |설정|값|
   |---|---|
   |사용자 이름|**Student**|
   |암호|**Pa55w.rd1234**|

   > **참고**: FSLogix 이진 파일을 설치하여 시작합니다.

1. az140-25-vm0에 대한 **Bastion 세션 내에서 Windows PowerShell ISE**를 관리자로 시작**** 합니다.
1. az140-25-vm0**에 대한 **Bastion 세션 내에서 관리istrator: Windows PowerShell ISE** 콘솔에서 **다음을 실행하여 이미지 구성을 위한 임시 위치로 사용할 폴더를 만듭니다.

   ```powershell
   New-Item -Type Directory -Path 'C:\Allfiles\Labs\02' -Force
   ```

1. az140-25-vm0에 대한 **Bastion 세션 내에서 Microsoft Edge를 시작하고, FSLogix 다운로드 페이지[로 이동하고](https://aka.ms/fslogix_download), FSLogix 압축 설치 이진 파일을 C:\\Allfiles\\Labs\\02** 폴더에 **다운로드하고, 파일 탐색기 x64** 하위 폴더를 동일한 폴더로 추출**** 합니다.
1. az140-25-vm0**에 대한 **Bastion 세션 내에서 관리istrator: Windows PowerShell ISE** 창으로 전환**하고 관리istrator: Windows PowerShell ISE** 콘솔에서 **다음을 실행하여 OneDrive의 컴퓨터별 설치를 수행합니다.

   ```powershell
   Start-Process -FilePath 'C:\Allfiles\Labs\02\x64\Release\FSLogixAppsSetup.exe' -ArgumentList '/quiet' -Wait
   ```

   > **참고**: 설치가 완료될 때까지 기다립니다. 약 1분 정도 걸릴 수 있습니다. 설치에서 다시 부팅을 트리거하는 경우 az140-25-vm0**에 다시 연결**합니다.

   > **참고**: 다음으로 Microsoft Teams의 설치 및 구성을 단계별로 진행합니다(이 랩에 사용된 이미지에 Teams가 이미 있으므로 학습 목적으로).

1. az140-25-vm0에 대한 **Bastion 세션 내에서 시작을** 마우스 오른쪽 단추로 클릭하고 **마우스 오른쪽 단추로 클릭 메뉴에서 실행을**** 선택하고 **실행** 대화 상자의 **열기** 텍스트 상자에 cmd**를 입력**하고 Enter** 키를 눌러 **명령 프롬프트**를 시작**합니다.**
1. **관리istrator: C:\windows\system32\cmd.exe** 창의 명령 프롬프트에서 다음을 실행하여 Microsoft Teams의 컴퓨터별 설치를 준비합니다.

   ```cmd
   reg add "HKLM\Software\Microsoft\Teams" /v IsWVDEnvironment /t REG_DWORD /d 1 /f
   ```

1. az140-25-vm0에 대한 **Bastion 세션 내에서 Microsoft Edge에서 Microsoft Visual C++ 재배포 가능 패키지[ 다운로드 페이지로 ](https://aka.ms/vs/16/release/vc_redist.x64.exe)이동하여 VC_redist.x64**를 C:\\Allfiles\\Labs\\02** 폴더에 **저장**합니다.**
1. Bastion 세션에서 **az140-25-vm0**으로 전환하고 **관리istrator: C:\windows\system32\cmd.exe** 창으로 전환하고 명령 프롬프트에서 다음을 실행하여 Microsoft Visual C++ 재배포 가능 패키지 설치를 수행합니다.

   ```cmd
   C:\Allfiles\Labs\02\vc_redist.x64.exe /install /passive /norestart /log C:\Allfiles\Labs\02\vc_redist.log
   ```

1. az140-25-vm0에 대한 **Bastion 세션 내에서 Microsoft Edge에서 VM[에 Teams 데스크톱 앱 배포라는 설명 ](https://docs.microsoft.com/en-us/microsoftteams/teams-for-vdi#deploy-the-teams-desktop-app-to-the-vm) 서 페이지로 이동하고, 64비트 버전** 링크를 클릭하고**, 메시지가 표시되면 Teams_windows_x64.msi** 파일을 C:\\Allfiles\\Labs\\02** 폴더에 **저장**합니다.**
1. Bastion 세션에서 **az140-25-vm0**으로 전환하고 **관리istrator: C:\windows\system32\cmd.exe** 창으로 전환하고 명령 프롬프트에서 다음을 실행하여 Microsoft Teams의 컴퓨터별 설치를 수행합니다.

   ```cmd
   msiexec /i C:\Allfiles\Labs\02\Teams_windows_x64.msi /l*v C:\Allfiles\Labs\02\Teams.log ALLUSER=1
   ```

   > **참고**: 설치 관리자는 ALLUSER=1 및 ALLUSERS=1 매개 변수를 지원합니다. ALLUSER=1 매개 변수는 VDI 환경에서 컴퓨터별 설치를 위한 것입니다. ALLUSERS=1 매개 변수는 비 VDI 및 VDI 환경에서 사용할 수 있습니다. 
   > **** 다른 버전의 제품이 이미 설치되어** 있다는 **오류가 발생하는 경우 다음 단계를 완료합니다. 제어판 > 프로그램 > 프로그램 및 기능**으로 이동하여 **Teams 컴퓨터 전체 설치 관리자** 프로그램을 마우스 오른쪽 단추로 클릭하고 **제거**를 선택합니다**. 프로그램 제거를 진행하고 위의 13단계를 다시 실행합니다. 

1. az140-25-vm0**에 대한 **Bastion 세션 내에서 Windows PowerShell ISE**를 관리istrator로 시작하고 **관리istrator: Windows PowerShell ISE** 콘솔에서 **다음을 실행하여 Microsoft Edge Chromium을 설치합니다(이 랩에 사용된 이미지에 Edge가 이미 있으므로 학습용).

   ```powershell
   Start-BitsTransfer -Source "https://aka.ms/edge-msi" -Destination 'C:\Allfiles\Labs\02\MicrosoftEdgeEnterpriseX64.msi'
   Start-Process -Wait -Filepath msiexec.exe -Argumentlist "/i C:\Allfiles\Labs\02\MicrosoftEdgeEnterpriseX64.msi /q"
   ```

   > **참고**: 설치가 완료될 때까지 기다립니다. 2분 정도 걸릴 수 있습니다.

   > **참고**: 다중 언어 환경에서 작동하는 경우 언어 팩을 설치해야 할 수 있습니다. 이 절차에 대한 자세한 내용은 Windows 10 다중 세션 이미지에 언어 팩 추가 Microsoft Docs 문서를 [참조하세요](https://docs.microsoft.com/en-us/azure/virtual-desktop/language-packs).

   > **참고**: 다음으로 Windows 자동 업데이트 사용하지 않도록 설정하고, Storage Sense를 사용하지 않도록 설정하고, 표준 시간대 리디렉션을 구성하고, 원격 분석 컬렉션을 구성합니다. 일반적으로 먼저 모든 현재 업데이트를 적용해야 합니다. 이 랩에서는 랩의 기간을 최소화하기 위해 이 단계를 건너뜁니다.

1. Bastion 세션에서 **az140-25-vm0**으로 전환하고 **관리istrator: C:\windows\system32\cmd.exe** 창으로 전환하고 명령 프롬프트에서 다음을 실행하여 자동 업데이트 사용하지 않도록 설정합니다.

   ```cmd
   reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU" /v NoAutoUpdate /t REG_DWORD /d 1 /f
   ```

1. **관리istrator: C:\windows\system32\cmd.exe** 창의 명령 프롬프트에서 다음을 실행하여 Storage Sense를 사용하지 않도록 설정합니다.

   ```cmd
   reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\StorageSense\Parameters\StoragePolicy" /v 01 /t REG_DWORD /d 0 /f
   ```

1. **관리istrator: C:\windows\system32\cmd.exe** 창의 명령 프롬프트에서 다음을 실행하여 표준 시간대 리디렉션을 구성합니다.

   ```cmd
   reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" /v fEnableTimeZoneRedirection /t REG_DWORD /d 1 /f
   ```

1. **관리istrator: C:\windows\system32\cmd.exe** 창의 명령 프롬프트에서 다음을 실행하여 원격 분석 데이터의 피드백 허브 컬렉션을 사용하지 않도록 설정합니다.

   ```cmd
   reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\DataCollection" /v AllowTelemetry /t REG_DWORD /d 0 /f
   ```

1. **관리istrator: C:\windows\system32\cmd.exe** 창의 명령 프롬프트에서 다음을 실행하여 이 작업의 앞부분에서 만든 임시 폴더를 삭제합니다.

   ```cmd
   rmdir C:\Allfiles /s /q
   ```

1. **관리istrator: C:\windows\system32\cmd.exe** 창의 명령 프롬프트에서 디스크 정리 유틸리티를 실행하고 완료되면 확인을** 클릭합니다**.

   ```cmd
   cleanmgr /d C: /verylowdisk
   ```

#### 작업 4: Azure Virtual Desktop 호스트 이미지 만들기

1. az140-25-vm0****에 대한 **Bastion 세션 내에서 관리istrator: C:\windows\system32\cmd.exe** 창의 명령 프롬프트에서 sysprep 유틸리티를 실행하여 이미지 생성을 위한 운영 체제를 준비하고 자동으로 종료합니다.

   ```cmd
   C:\Windows\System32\Sysprep\sysprep.exe /oobe /generalize /shutdown /mode:vm
   ```

   > **참고**: sysprep 프로세스가 완료되기를 기다립니다. 2분 정도 걸릴 수 있습니다. 그러면 운영 체제가 자동으로 종료됩니다. 

1. 랩 컴퓨터의 Azure Portal을 표시하는 웹 브라우저에서 가상 머신을 검색하여 선택하고 **가상 머신**** 블레이드에서 **az140-25-vm0**을 선택합니다**.
1. **az140-25-vm0** 블레이드의 Essentials** 섹션 위의 **도구 모음에서 새로 고침**을 클릭하고**, Azure VM의 상태가** 중지됨으로 **변경되었는지** 확인하고**, 중지**를 클릭하고**, 확인 메시지가 표시되면 확인을** 클릭하여 **Azure VM**을 중지됨(할당 취소됨)** 상태로 전환합니다.
1. az140-25-vm0** 블레이드에서 Azure VM의 상태가** 중지됨(할당 취소됨)** 상태로 변경**되었는지 확인하고 **도구 모음에서 캡처**를 클릭합니다**.** 그러면 이미지** 만들기 블레이드가 **자동으로 표시됩니다.
1. **이미지** 만들기 블레이드의 기본 사항 탭에서 **다음 설정을 지정합니다**.

   |설정|값|
   |---|---|
   |Azure Compute Gallery로 이미지 공유|**예, 이미지 버전으로 갤러리에 공유합니다.**|
   |이미지를 만든 후 이 가상 머신을 자동으로 삭제|검사box가 지워진 경우|
   |대상 Azure Compute Gallery|새 이미지 갤러리 **az14025imagegallery**의 이름|
   |운영 체제 상태|**일반화**|

1. **이미지 만들기** 블레이드의 **기본** 탭에서 **대상 VM 이미지 정의** 텍스트 상자 아래의 **새로 만들기**를 클릭합니다.
1. **VM 이미지 정의 만들기**에서 다음 설정을 지정하고 **확인**을 클릭합니다.

   |설정|값|
   |---|---|
   |VM 이미지 정의 이름|**az140-25-host-image**|
   |게시자|**MicrosoftWindowsDesktop**|
   |제안|**office-365**|
   |SKU|**win11-22h2-avd-m365**|

1. 이미지 만들기 블레이드의 **기본 사항** 탭으로 돌아가서 다음 설정을 지정하고 검토 + 만들기**를 클릭합니다**.** ** 

   |설정|값|
   |---|---|
   |버전 번호|**1.0.0**|
   |최신에서 제외|검사box가 지워진 경우|
   |수명 종료 날짜|현재 날짜보다 1년 앞당기기|
   |기본 복제본(replica) 개수|**1**|
   |대상 지역 복제본(replica) 수|**1**|
   |Storage 계정 유형|**프리미엄 SSD LRS**|

1. **이미지** 만들기 블레이드의 검토 + 만들기** 탭에서 **만들기**를 클릭합니다**.

   > **참고**: 배포가 완료될 때까지 기다립니다. 20분 정도 걸릴 수 있습니다.

1. 랩 컴퓨터의 Azure Portal이 표시된 웹 브라우저에서 **Azure Compute Galleries**를 검색하여 선택하고 **Azure Compute Galleries** 블레이드에서 **az14025imagegallery** 항목을 선택합니다. 그런 다음, ****az10425imagegallery**** 블레이드에서 새로 만든 이미지에 해당하는 **az140-25-host-image** 항목이 있는지 확인합니다.

#### 작업 5: 사용자 지정 이미지를 사용하여 Azure Virtual Desktop 호스트 풀 프로비전

1. 랩 컴퓨터의 Azure Portal에서 Azure Portal 페이지의 맨 위에 있는 검색 리소스, 서비스 및 문서 텍스트 상자를 사용하여 **가상 네트워크를 검색 및 탐색**하고 가상 네트워크**** 블레이드에서 **az140-adds-vnet11**을 선택합니다**.** 
1. az140-adds-vnet11** 블레이드에서 서브넷을 선택하고 **, 서브넷** 블레이드에서 **+ 서브넷**을 선택하고 **, 서브넷**** 추가 블레이드에서 **다음 설정을 지정하고(다른 모든 설정을 기본값으로 유지) 저장**을 클릭합니다**.**

   |설정|값|
   |---|---|
   |속성|**hp4-Subnet**|
   |서브넷 주소 범위|**10.0.4.0/24**|

1. 랩 컴퓨터의 Azure Portal이 표시된 웹 브라우저 창에서 **Azure Virtual Desktop**을 검색하여 선택하고 **Azure Virtual Desktop** 블레이드에서 **호스트 풀**을 선택하고 **Azure Virtual Desktop \| 호스트 풀** 블레이드에서 **+ 만들기**를 선택합니다. 
1. 호스트 풀** 만들기 블레이드의 **기본 사항** 탭에서 다음 설정을 지정하고 다음을 선택합니다 **. Virtual Machines >**.**

   |설정|값|
   |---|---|
   |구독|이 랩에서 사용 중인 Azure 구독의 이름|
   |Resource group|**az140-25-RG**|
   |호스트 풀 이름|**az140-25-hp4**|
   |위치|이 랩의 첫 번째 연습에서 리소스를 배포한 Azure 지역의 이름|
   |유효성 검사 환경|**문제**|
   |호스트 풀 유형|**풀링된**|
   |최대 세션 제한|**50**|
   |부하 분산 알고리즘|**너비 우선**|

1. **호스트 풀** 만들기 블레이드의 가상 머신** 탭에서 **다음 설정을 지정합니다.

   |설정|값|
   |---|---|
   |Azure 가상 머신 추가|**예**|
   |Resource group|**호스트 풀과 같은 그룹으로 기본 지정됨**|
   |이름 접두사|**az140-25-p4**|
   |가상 머신 위치|이 랩의 첫 번째 연습에서 리소스를 배포한 Azure 지역의 이름|
   |가용성 옵션|**인프라 중복 필요 없음**|
   
1. **호스트 풀** 만들기 블레이드의 **가상 머신** 탭에서 이미지** 드롭다운 목록 바로 아래에 있는 ****모든 이미지** 보기 링크를 클릭합니다.
1. 이미지 선택 블레이드의 **다른 항목**에서 **공유 이미지를** 클릭하고 **공유 이미지 목록에서 az140-25-host-image**를 선택합니다**.** 
1. **호스트 풀** 만들기 블레이드의 **가상 머신** 탭으로 돌아가서 다음 설정을 지정하고 다음을 선택합니다 **. 작업 영역 >**

   |설정|값|
   |---|---|
   |가상 머신 크기|**표준 D2s v3**|
   |VM 수|**1**|
   |OS 디스크 유형|**표준 SSD**|
   |가상 네트워크|**az140-adds-vnet11**|
   |서브넷|**hp4-Subnet(10.0.4.0/24)**|
   |네트워크 보안 그룹|**기본**|
   |퍼블릭 인바운드 포트|**예**|
   |허용할 인바운드 포트|**RDP**|
   |AD do기본 JOIN UPN|**student@adatum.com**|
   |암호|**Pa55w.rd1234**|
   |do기본 또는 unit 지정|**예**|
   |가입할 도메인|**adatum.com**|
   |조직 구성 단위 경로|**OU=WVDInfra,DC=adatum,DC=com**|
   |사용자 이름|Student|
   |암호|Pa55w.rd1234|
   |암호 확인|Pa55w.rd1234|

1. 호스트 풀** 만들기 블레이드의 작업 영역** 탭에서 **다음 설정을 지정하고 검토 + 만들기**를 선택합니다**.**

   |설정|값|
   |---|---|
   |데스크톱 앱 그룹 등록|**문제**|

1. **호스트 풀** 만들기 블레이드의 **검토 + 만들기** 탭에서 만들기**를 선택합니다**.

   > **참고**: 배포가 완료될 때까지 기다립니다. 10분 정도 걸릴 수 있습니다.
   > 
   > **참고** 할당량 한도에 도달하여 배포에 실패한 경우 첫 번째 랩에 설명된 단계를 수행하여 표준 D2sv3 한도를 30으로 자동으로 증가하도록 요청합니다.

   > **참고**: 사용자 지정 이미지를 기반으로 호스트를 배포한 후에는 GitHub 리포지[토리에서 ](https://github.com/The-Virtual-Desktop-Team/)사용할 수 있는 가상 데스크톱 최적화 도구를 실행하는 것이 좋습니다.


### 연습 2: 랩에서 프로비전된 Azure VM 중지 및 할당 취소

이 연습의 주요 작업은 다음과 같습니다.

1. 랩에서 프로비전된 Azure VM 중지 및 할당 취소

>**참고**: 이 연습에서는 이 랩에 프로비전된 Azure VM의 할당을 취소하여 해당 컴퓨팅 요금을 최소화합니다.

#### 작업 1: 랩에서 프로비전된 Azure VM 할당 취소

1. 랩 컴퓨터로 전환하고 Azure Portal을 표시하는 웹 브라우저 창에서 Cloud Shell 창 내에서 **PowerShell** 셸** 세션을 엽니다**.
1. Cloud Shell 창의 PowerShell 세션에서 다음을 실행하여 이 랩에서 만든 모든 Azure VM을 나열합니다.

   ```powershell
   Get-AzVM -ResourceGroup 'az140-25-RG'
   ```

1. Cloud Shell 창의 PowerShell 세션에서 다음을 실행하여 이 랩에서 만든 모든 Azure VM을 중지하고 할당을 취소합니다.

   ```powershell
   Get-AzVM -ResourceGroup 'az140-25-RG' | Stop-AzVM -NoWait -Force
   ```

   >**참고**: 명령은 -NoWait 매개 변수에 의해 결정된 대로 비동기적으로 실행되므로 동일한 PowerShell 세션 내에서 즉시 다른 PowerShell 명령을 실행할 수 있지만 Azure VM이 실제로 중지되고 할당이 취소되기까지 몇 분 정도 걸립니다.
