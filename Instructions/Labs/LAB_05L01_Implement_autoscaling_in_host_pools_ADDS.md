---
lab:
  title: '랩: 호스트 풀에서 자동 크기 조정 구현(AD DS)'
  module: 'Module 5: Monitor and Maintain an AVD Infrastructure'
---

# 랩 - 호스트 풀에서 자동 크기 조정 구현(AD DS)
# 학생용 랩 매뉴얼

## 랩 종속성

- 이 랩에서 사용할 Azure 구독입니다.
- 이 랩에서 사용할 Azure 구독의 소유자 또는 기여자 역할이 있는 Microsoft 계정 또는 Microsoft Entra 계정 및 해당 Azure 구독과 연결된 Microsoft Entra 테넌트의 Global 관리istrator 역할.
- 완료된 **Azure Virtual Desktop의 배포 준비(AD DS)** 랩
- 완료된 랩 **은 AD DS(Azure Portal)를 사용하여 호스트 풀 및 세션 호스트를 배포합니다.**

## 예상 소요 시간

60분

## 실습 시나리오

AD DS(Active Directory 도메인 Services) 환경에서 Azure Virtual Desktop 세션 호스트의 자동 크기 조정을 구성해야 합니다.

## 목표
  
이 랩을 완료하면 다음을 수행할 수 있습니다.

- Azure Virtual Desktop 세션 호스트의 자동 크기 조정 구성
- Azure Virtual Desktop 세션 호스트의 자동 크기 조정 확인

## 랩 파일

- 없음

## 지침

### 연습 1: Azure Virtual Desktop 세션 호스트의 자동 크기 조정 구성

이 연습의 주요 작업은 다음과 같습니다.

1. Azure Virtual Desktop 세션 호스트 크기 조정 준비
2. Azure Virtual Desktop 세션 호스트에 대한 크기 조정 계획 만들기

#### 작업 1: Azure Virtual Desktop 세션 호스트 크기 조정 준비

1. 랩 컴퓨터에서 웹 브라우저를 시작하고, Azure Portal[로 ](https://portal.azure.com)이동한 다음, 이 랩에서 사용할 구독에서 소유자 역할이 있는 사용자 계정의 자격 증명을 제공하여 로그인합니다.
1. 랩 컴퓨터의 Azure Portal을 표시하는 웹 브라우저 창에서 Cloud Shell 창에서 **PowerShell** 세션을 엽니다**.**

   >**참고**: 자동 크기 조정과 함께 사용하려는 호스트 풀은 MaxSessionLimit** 매개 변수의 기본값이 **아닌 값으로 구성해야 합니다. 이 예제와 같이 Azure Portal의 호스트 풀 설정에서 또는 Update-AzWvdHostPool** Azure PowerShell cmdlet을 실행**하여 이 값을 설정할 수 있습니다. Azure Portal에서 풀을 만들거나 New-AzWvdHostPool** Azure PowerShell cmdlet을 **실행할 때 명시적으로 설정할 수도 있습니다.

1. Cloud Shell 창의 PowerShell 세션에서 다음 명령을 실행하여 az140-21-hp1** 호스트 풀의 **** MaxSessionLimit** 매개 변수 값을 2**로 **설정합니다. 

   ```powershell
   Update-AzWvdHostPool -ResourceGroupName 'az140-21-RG' `
   -Name az140-21-hp1 `
   -MaxSessionLimit 2
   ```

   >**참고**: 이 랩에서는 자동 크기 조정 동작을 **쉽게 트리거하기 위해 MaxSessionLimit** 매개 변수의 값이 인위적으로 낮게 설정됩니다.

   >**참고**: 첫 번째 크기 조정 계획을 만들기 전에 Azure 구독을 대상 범위로 사용하여 Desktop Virtualization Power On Off 기여자** RBAC 역할을 Azure Virtual Desktop에 할당**해야 합니다. 

1. Azure Portal을 표시하는 브라우저 창에서 Cloud Shell 창을 닫습니다.
1. Azure Portal에서 구독**을 검색하여 선택하고 **구독 목록에서 Azure Virtual Desktop 리소스가 포함된 구독을 선택합니다. 
1. 구독 페이지에서 액세스 제어(IAM)**를 선택합니다**.
1. **액세스 제어(IAM)** 페이지의 도구 모음에서 + 추가 단추를** 선택한 **다음, 드롭다운 메뉴에서 역할 할당** 추가를 선택합니다**.
1. **역할 할당** 추가 마법사의 **역할** 탭에서 Desktop Virtualization Power On Off 기여자** 역할을 검색하여 선택하고 **다음**을 클릭합니다**.
1. 역할 할당 추가 마법사의 **구성원** 탭에서 + 멤버**를 선택하고**, Azure Virtual Desktop 또는 **Windows Virtual Desktop**** 을 검색하여 선택하고**, [선택 **]을 클릭하고 **[다음 **]을 클릭합니다**.** ** 

   >**참고**: 값은 Microsoft.DesktopVirtualization** 리소스 공급자가 Azure 테넌트에 처음 등록된 시기에 **따라 달라집니다.

1. 검토 + 할당 탭에서 **검토 + 할당**을 선택합니다**.**

#### 작업 2: Azure Virtual Desktop 세션 호스트에 대한 크기 조정 계획 만들기

1. 랩 컴퓨터의 브라우저에서 Azure Portal을 표시하고 Azure Virtual Desktop**을 검색하여 선택합니다**. 
1. **Azure Virtual Desktop** 페이지에서 크기 조정 계획을** 선택한 **다음+ 만들기**를 선택합니다**.
1. **크기 조정 계획** 만들기 마법사의 **기본** 사항 탭에서 다음 정보를 지정하고 다음 일정 >** 선택합니다**(다른 사용자는 기본값을 그대로 둡니다.)

   |설정|값|
   |---|---|
   |Resource group|새 리소스 그룹의 az140-51-RG** 이름 **|
   |이름|**az140-51-scaling-plan**|
   |위치|이전 랩에서 세션 호스트를 배포한 동일한 Azure 지역|
   |식별 이름|**az140-51 크기 조정 계획**|
   |Time zone|현지 표준 시간대|

   >**참고**: 제외 태그를 사용하면 크기 조정 작업에서 제외하려는 세션 호스트의 태그 이름을 지정할 수 있습니다. 예를 들어 제외 태그 “excludeFromScaling”을 사용하여 유지 관리 중에 자동 크기 조정이 드레이닝 모드를 재정의하지 않도록 드레이닝 모드로 설정된 VM에 태그를 지정할 수 있습니다. 

1. **크기 조정 계획** 만들기 마법사의 일정** 탭에서 **+ 일정** 추가를 선택합니다**.
1. **일정** 추가 마법사의 일반** 탭에서 **다음 정보를 지정하고 다음**을 클릭합니다**.

   |설정|값|
   |---|---|
   |일정 이름|**az140-51-schedule**|
   |반복|**7개** 선택됨(요일 모두 선택)|

1. **일정** 추가 마법사의 **램프업** 탭에서 다음 정보를 지정하고 다음**을 클릭합니다**.

   |설정|값|
   |---|---|
   |시작 시간(24시간 시스템)|현재 시간 -9시간|
   |부하 분산 알고리즘|**첫 번째 폭**|
   |호스트의 최소 백분율(%)|**20**|
   |용량 임계값(%)|**60**|

   >**참고**: 여기에서 선택한 부하 분산 기본 설정은 원래 호스트 풀 설정에 대해 선택한 기본 설정을 재정의합니다.

   >**참고**: 호스트의 최소 비율은 항상 다시 사용하려는 세션 호스트의 비율을 지정합니다기본. 입력한 백분율이 정수가 아니면 가장 가까운 정수로 반올림됩니다. 

   >**참고**: 용량 임계값은 수행될 크기 조정 작업을 트리거하는 사용 가능한 호스트 풀 용량의 비율을 나타냅니다. 예를 들어 최대 세션 제한이 20인 호스트 풀의 2개의 세션 호스트가 켜져 있는 경우 사용 가능한 호스트 풀 용량은 40입니다. 용량 임계값을 75%로 설정하고 세션 호스트에 30개 이상의 사용자 세션이 있는 경우 자동 크기 조정은 세 번째 세션 호스트를 켭니다. 그러면 사용 가능한 호스트 풀 용량이 40에서 60으로 변경됩니다.

1. **일정** 추가 마법사의 사용 시간** 탭에서 **다음 정보를 지정하고 다음**을 클릭합니다**.

   |설정|값|
   |---|---|
   |시작 시간(24시간 시스템)|현재 시간 -8시간|
   |부하 분산 알고리즘|**깊이 우선**|

   >**참고**: 시작 시간은 램프업 단계의 종료 시간을 지정합니다.

   >**참고**: 이 단계의 용량 임계값은 램프 업 용량 임계값에 따라 결정됩니다.

1. **일정** 추가 마법사의 **램프다운** 탭에서 다음 정보를 지정하고 다음**을 클릭합니다**.

   |설정|값|
   |---|---|
   |시작 시간(24시간 시스템)|현재 시간 -2시간|
   |부하 분산 알고리즘|**깊이 우선**|
   |호스트의 최소 백분율(%)|**10**|
   |용량 임계값(%)|**90**|
   |사용자 강제 로그오프|**예**|
   |사용자를 로그아웃하고 VM을 종료하기 전의 지연 시간(분)|**30**|

   >**참고**: 강제 로그오프 사용자를** 사용하도록 설정한 경우 **자동 크기 조정은 사용자 세션 수가 가장 낮은 세션 호스트를 드레이닝 모드로 설정하고, 모든 활성 사용자 세션에 임박한 종료에 대한 알림을 보내고, 지정된 지연 시간이 지나면 강제로 로그아웃합니다. 자동 크기 조정이 모든 사용자 세션을 로그아웃한 후 VM의 할당을 취소합니다. 

   >**참고**: 램프 다운 중에 강제 로그아웃을 사용하도록 설정하지 않은 경우 활성 또는 연결이 끊긴 세션이 없는 세션 호스트의 할당이 취소됩니다.

1. **일정** 추가 마법사의 **사용량이 많은 시간** 탭에서 다음 정보를 지정하고 추가**를 클릭합니다**.

   |설정|값|
   |---|---|
   |시작 시간(24시간 시스템)|현재 시간 -1시간|
   |부하 분산 알고리즘|**깊이 우선**|

   >**참고**: 이 단계의 용량 임계값은 램프 다운 용량 임계값에 의해 결정됩니다.

1. **크기 조정 계획** 만들기 마법사의 **일정** 탭으로 돌아가서 다음: 호스트 풀 할당 >** 선택합니다**.
1. **호스트 풀 할당** 페이지의 **호스트 풀** 선택 드롭다운 목록에서 az140-21-hp1**을 선택하고 **자동 크기 조정** 사용 검사 상자가 사용하도록 설정되어 있는지 확인하고 **검토 + 만들기**를 선택한 **다음 만들기**를 선택합니다**.


### 연습 2: Azure Virtual Desktop 세션 호스트의 자동 크기 조정 확인

이 연습의 주요 작업은 다음과 같습니다.

1. Azure Virtual Desktop 자동 크기 조정을 추적하도록 진단 설정
1. Azure Virtual Desktop 세션 호스트의 자동 크기 조정 확인

#### 작업 1: Azure Virtual Desktop 자동 크기 조정을 추적하도록 진단 설정

1. 랩 컴퓨터의 Azure Portal을 표시하는 웹 브라우저 창에서 Cloud Shell 창에서 **PowerShell** 세션을 엽니다**.**

   >**참고**: Azure Storage 계정을 사용하여 자동 크기 조정 이벤트를 저장합니다. 이 작업에 설명된 대로 Azure Portal에서 직접 만들거나 Azure PowerShell을 사용할 수 있습니다.

1. Cloud Shell 창의 PowerShell 세션에서 다음 명령을 실행하여 Azure Storage 계정을 만듭니다.

   ```powershell
   $resourceGroupName = 'az140-51-RG'
   $location = (Get-AzResourceGroup -ResourceGroupName $resourceGroupName).Location
   $suffix = Get-Random
   $storageAccountName = "az140st51$suffix"
   New-AzStorageAccount -Location $location -Name $storageAccountName -ResourceGroupName $resourceGroupName -SkuName Standard_LRS
   ```

   >**참고**: 스토리지 계정이 프로비전될 때까지 기다립니다.

1. Azure Portal을 표시하는 브라우저 창에서 Cloud Shell 창을 닫습니다.
1. 랩 컴퓨터의 Azure Portal을 표시하는 브라우저에서 이전 연습에서 만든 크기 조정 계획의 페이지로 이동합니다.
1. az140-51-scaling-plan** 페이지에서 진단 설정을** 선택한 **다음+ 진단 설정** 추가를 선택합니다**.**
1. **진단 설정** 페이지의 **진단 설정 이름** 텍스트 상자에 az140-51-scaling-plan-진단** 입력**하고 범주 **그룹** 섹션에서 allLogs**를 선택합니다**. 
1. 같은 페이지의 **대상 세부 정보** 섹션에서 스토리지 계정에 보관을 선택하고 **스토리지 계정**** **드롭다운 목록에서 az140st51** 접두사로 시작하는 **스트러지 계정 이름을 선택합니다.
1. **저장**을 선택합니다.

#### 작업 2: Azure Virtual Desktop 세션 호스트의 자동 크기 조정 확인

1. 랩 컴퓨터의 Azure Portal을 표시하는 웹 브라우저 창에서 Cloud Shell 창에서 **PowerShell** 세션을 엽니다**.**
1. Cloud Shell 창의 PowerShell 세션에서 다음 명령을 실행하여 이 랩에서 사용할 Azure Virtual Desktop 세션 호스트 Azure VM을 시작합니다.

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Start-AzVM
   ```

   >**참고**: 세션 호스트 Azure VM이 실행될 때까지 기다립니다.

1. 랩 컴퓨터의 Azure Portal을 표시하는 웹 브라우저 창에서 az140-21-hp1** 호스트 풀의 **페이지로 이동합니다.
1. **az140-21-hp1** 페이지에서 세션 호스트를** 선택합니다**.
1. 종료** 상태 하나 이상의 세션 호스트가 **나열될 때까지 기다립니다.

   >**참고**: 세션 호스트의 상태 업데이트하려면 페이지를 새로 고쳐야 할 수 있습니다.

   >**참고**: 모든 세션 호스트를 다시 사용할 수 기본 az140-51-scaling-plan** 페이지로 돌아가**서 최소 호스트 비율(%)** **램프 다운** 설정의 **값을 줄입니다.

   >**참고**: 하나 이상의 세션 호스트 상태 변경되면 Azure Storage 계정에서 자동 크기 조정 로그를 사용할 수 있어야 합니다. 

1. Azure Portal에서 Storage 계정을** 검색하고 선택하고 **Storage **계정** 페이지에서 이 연습의 앞부분에서 만든 스토리지 계정을 나타내는 항목을 선택합니다(이름은 az140st51** 접두사로 **시작).
1. 스토리지 계정 페이지에서 컨테이너를** 선택합니다**.
1. 컨테이너 목록에서 insights-logs-autoscale**을 선택합니다**.
1. **insights-logs-autoscale** 페이지에서 컨테이너에 저장된 JSON 형식 Blob을 나타내는 항목에 도달할 때까지 폴더 계층 구조를 탐색합니다.
1. Blob 항목을 선택하고 페이지 맨 오른쪽에 있는 줄임표 아이콘을 선택한 다음 드롭다운 메뉴에서 다운로드**를 선택합니다**.
1. 랩 컴퓨터에서 선택한 텍스트 편집기에서 다운로드한 Blob을 열고 해당 콘텐츠를 검사합니다. 자동 크기 조정 이벤트에 대한 참조를 찾을 수 있어야 합니다. 

   >**참고**: 다음은 자동 크기 조정 이벤트에 대한 참조를 포함하는 샘플 Blob 콘텐츠입니다.

   ```json
   host_Ring    "R0"
   Level    4
   ActivityId   "00000000-0000-0000-0000-000000000000"
   time "2023-03-26T19:35:46.0074598Z"
   resourceId   "/SUBSCRIPTIONS/AAAAAAAE-0000-1111-2222-333333333333/RESOURCEGROUPS/AZ140-51-RG/PROVIDERS/MICROSOFT.DESKTOPVIRTUALIZATION/SCALINGPLANS/AZ140-51-SCALING-PLAN"
   operationName    "ScalingEvaluationSummary"
   category "Autoscale"
   resultType   "Succeeded"
   level    "Informational"
   correlationId    "ddd3333d-90c2-478c-ac98-b026d29e24d5"
   properties   
   Message  "Active session hosts are at 0.00% capacity (0 sessions across 3 active session hosts). This is below the minimum capacity threshold of 90%. 2 session hosts can be drained and deallocated."
   HostPoolArmPath  "/subscriptions/aaaaaaaa-0000-1111-2222-333333333333/resourcegroups/az140-21-rg/providers/microsoft.desktopvirtualization/hostpools/az140-21-hp1"
   ScalingEvaluationStartTime   "2023-03-26T19:35:43.3593413Z"
   TotalSessionHostCount    "3"
   UnhealthySessionHostCount    "0"
   ExcludedSessionHostCount "0"
   ActiveSessionHostCount   "3"
   SessionCount "0"
   CurrentSessionOccupancyPercent   "0"
   CurrentActiveSessionHostsPercent "100"
   Config.ScheduleName  "az140-51-schedule"
   Config.SchedulePhase "OffPeak"
   Config.MaxSessionLimitPerSessionHost "2"
   Config.CapacityThresholdPercent  "90"
   Config.MinActiveSessionHostsPercent  "5"
   DesiredToScaleSessionHostCount   "-2"
   EligibleToScaleSessionHostCount  "1"
   ScalingReasonType    "DeallocateVMs_BelowMinSessionThreshold"
   BeganForceLogoffOnSessionHostCount   "0"
   BeganDeallocateVmCount   "1"
   BeganStartVmCount    "0"
   TurnedOffDrainModeCount  "0"
   TurnedOnDrainModeCount   "1"
   ```


### 연습 3: 랩에서 프로비전된 Azure VM 중지 및 할당 취소

이 연습의 주요 작업은 다음과 같습니다.

1. 랩에서 프로비전된 Azure VM 중지 및 할당 취소

>**참고**: 이 연습에서는 이 랩에서 사용되는 Azure VM의 할당을 취소하여 해당 컴퓨팅 요금을 최소화합니다.

#### 작업 1: 랩에서 프로비전된 Azure VM 할당 취소

1. 랩 컴퓨터로 전환하고 Azure Portal을 표시하는 웹 브라우저 창에서 Cloud Shell 창에서 **PowerShell** 세션을 엽니다**.**
1. Cloud Shell 창의 PowerShell 세션에서 다음 명령을 실행하여 이 랩에서 사용되는 모든 Azure VM을 나열합니다.

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG'
   ```

1. Cloud Shell 창의 PowerShell 세션에서 다음 명령을 실행하여 이 랩에서 사용한 모든 Azure VM을 중지하고 할당을 취소합니다.

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Stop-AzVM -NoWait -Force
   ```

   >**참고**: 명령은 -NoWait 매개 변수에 의해 결정된 대로 비동기적으로 실행되므로 동일한 PowerShell 세션 내에서 즉시 다른 PowerShell 명령을 실행할 수 있지만 Azure VM이 실제로 중지되고 할당이 취소되기까지 몇 분 정도 걸립니다.
