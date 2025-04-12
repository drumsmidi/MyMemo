# RunbookのPowerShellによるAppService起動・停止サンプル

## Runbook

- Automationサービスより、[プロセスオートメーション]-[Runbook の作成]より作成  
　スクリプト入力後、[テストウィンドウ]で実行後、[保存][公開]と実施する必要がありそう。  

- [スケジュールのリンク]より、実行スケジュールを登録する  

## 起動

```PowerShell
$ServicePlanName = ""
$Sku = "P1mv3"
$ResourceGroupName = ""
$AppServiceNames = @(
  "",
  "",
  ""
)
$SlotNames = @(
  "production",
  "dev"
)

$returnCode = 0

# AzureにマネージドIDで接続
Connect-AzAccount -Identity

# App Service Plan スケールアップ
try {
  $servicePlanConf = Get-AzAppServicePlan -ResourceGroupName $ResourceGroupName -Name $ServicePlanName

  if ($servicePlanConf.Sku.Size -eq $Sku) {
    Write-Output "Success: App Service Plan already $Sku"
  } else {
    $servicePlanConf.Sku.Name = $Sku
    $servicePlanConf.Sku.Size = $Sku
    $servicePlanConf | Set-AzAppServicePlan
    Write-Output "Success: App Service Plan scaled up to $Sku"
  }
} catch {
  Write-Error "Failure: App Service Plan scaled up $_"
  $returnCode = 1
}

if ($returnCode -eq 0) {
  # App Serviceを開始
  foreach ($appservicename in $AppServiceNames) {
    foreach ($slotname in $SlotNames) {
      try {
        Start-AzWebAppSlot -ResourceGroupName $ResourceGroupName -Name $appservicename -Slot $slotname
        Write-Output "Success: App Service Start: $appservicename - $slotname"
      } catch {
        Write-Output "Failure: App Service Start: $appservicename - $slotname"
        $returnCode = 2
      }
    }
  }
}

exit $returnCode
```

## 停止

```PowerShell
$ServicePlanName = ""
$Sku = "P1mv3"
$ResourceGroupName = ""
$AppServiceNames = @(
  "",
  "",
  ""
)
$SlotNames = @(
  "production",
  "dev"
)

$returnCode = 0

# AzureにマネージドIDで接続
Connect-AzAccount -Identity

# App Serviceを停止
foreach ($appservicename in $AppServiceNames) {
  foreach ($slotname in $SlotNames) {
    try {
      Stop-AzWebAppSlot -ResourceGroupName $ResourceGroupName -Name $appservicename -Slot $slotname
      Write-Output "Success: App Service Stop: $appservicename - $slotname"
    } catch {
      Write-Output "Failure: App Service Stop: $appservicename - $slotname"
      $returnCode = 1
    }
  }
}

if ($returnCode -eq 0) {
  # App Service Plan スケールダウン
  try {
    $servicePlanConf = Get-AzAppServicePlan -ResourceGroupName $ResourceGroupName -Name $ServicePlanName

    if ($servicePlanConf.Sku.Size -eq $Sku) {
      Write-Output "Success: App Service Plan already $Sku"
    } else {
      $servicePlanConf.Sku.Name = $Sku
      $servicePlanConf.Sku.Size = $Sku
      $servicePlanConf | Set-AzAppServicePlan
      Write-Output "Success: App Service Plan scaled up to $Sku"
    }
  } catch {
    Write-Error "Failure: App Service Plan scaled up $_"
    $returnCode = 2
  }
}

exit $returnCode
```
