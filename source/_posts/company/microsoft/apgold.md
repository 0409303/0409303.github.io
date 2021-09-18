d:\project\APGold\autopilotservice\Global\VirtualEnvironments\AdsBI\AdsMz-Test-MW1-MSNMediation>sd edit deployment.int
deployment.int - file(s) not on client.


直接编辑




1. add/edit
2. 安装codeflow 提交pr
\\codeflow.redmond.corp.microsoft.com\public\cf2Launcher.cmd

codeflow作用



ve作用：
pe作用：

之前



D:\project\APGold\autopilotservice\Global\VirtualEnvironments\AdsBI\AdsMz-Test-MW1-MSNMediation

D:\project\APGold\autopilotservice\MW1\AdsMz-Prod-MW1


A ServiceGroup is mandatory for every machine function



VE中存储的可以理解为某个项目的global config，所有的PE公用这部分config
设计目的是在同一项目部署多个PE时，避免多次发布，只需要更新VE，相关PE就会生效





https://msazure.visualstudio.com



D:\project\APGold\autopilotservice\Global\VirtualEnvironments\AdsBI\AdsMz-Prod-MW1-MSNMediation







```
07/26/21 16:54:01.841,error processing AdsMz-Test-MW1: Rollout 'AdsBI-AdsMz-Test-MW1-MzOrchestration-VE.6799404_17553440741849145769_0.Rollout_AdsBI_Streaming_CFR' cannot be kicked due to status: InRollout - Rollout in progress..Rollout 'AdsBI-AdsMz-Test-MW1-MSNMediation-VE.6827694_4064931696698155710_0.Rollout_AdsBI_Streaming_CFR' cannot be kicked due to status: OtherRollout - Waiting on other rollout on MF AdsBI_Streaming_CFR in progress: AdsBI_Streaming_CFR.AdvertiserAggs_94d0ef91_90546_1_15-0-897+94d0ef91_CL6670088_5920165827906938206_0.AdsBI-AdsMz-Test-MW1-AdvertiserAggs-VE.csv:AdsBI_Streaming_CFR.AgoraAggs_76700988_85934_1_16-0-173+76700988_CL6670088_4056715873563314682_0.AdsBI-AdsMz-Test-MW1-AgoraAggs-VE.csv:AdsBI_Streaming_CFR.CACFR_9a63868e_90143_2_1-0-2408+9a63868e_CL6670088_11167218336511992544_0.AdsBI-AdsMz-Test-MW1-CACFR-VE.csv:AdsBI_Streaming_CFR.CFR_2d2cea68_91067_1_17-0-6662+2d2cea68_CL6670088_18415503406869780426_0.AdsBI-AdsMz-Test-MW1-CFR-VE.csv:AdsBI_Streaming_CFR.KpiAggs_fdd5f92d_90180_1_17-0-433+fdd5f92d_CL6670088_6626197140932378707_0.AdsBI-AdsMz-Test-MW1-KPIAggs-VE.csv:AdsBI_Streaming_CFR.Monetization_de0db608_91071_1_1-0-2808+de0db608_CL6670088_351842135959690630_0.AdsBI-AdsMz-Test-MW1-Monetization-VE.csv:AdsBI_Streaming_CFR.Orchestration_93eb645e_91078_1_16-0-557+93eb645e_CL6799404_17553440741849145769_0.AdsBI-AdsMz-Test-MW1-MzOrchestration-VE.csv:AdsBI_Streaming_CFR.PublisherAggs_55bd6dd9_90534_1_17-0-474+55bd6dd9_CL6670088_15361642287162420254_0.AdsBI-AdsMz-Test-MW1-PublisherAggs-VE.csv:AdsBI_Streaming_CFR.Mediation_865d3b39_90170_1_merge_20210629_-1_CL6670088_3189803787641037971_0.AdsBI-AdsMz-Test-MW1-Mediation-VE.csv:AdsBI_Streaming_CFR.MSNMediation_4a63e32e_91077_2_0-1-28+4a63e32e33_CL6827694_4798040603905411799_0.AdsBI-AdsMz-Test-MW1-MSNMediation-VE.csv:.OM.Autopilot-AutopilotClient-VE.csv..; .. Some rollouts not triggered.
```





```
Deployment: 6827694_4798040603905411799 (this is NOT the latest deployment.ini change number)
Downloaded 89 files, 21510455 bytes, compressed size: 8267353 bytes.
Failed to build services: Failed to build service 'SAMHourlyS2B.MSNMediation_4a63e32e_91077_2_0-1-28+4a63e32e33': Proxy error: not all data was received (DP: 25.68.114.53; by MW01NAP0000036C); ;  ;Log:e:GetStreamChunkMapWithRetry:stream does not exist [response from server] : GetStreamChunkMap('stream://AdsBI-AdsMz-Test-MW1-MSNMediation-VE/app/ServiceMaps/MSNMediationServiceMap.ini'; 'MSNMediation_4a63e32e_91077_2_0-1-28+4a63e32e33') failed ;Log:w:ApDynamicStorage::StorageConnectionEx::DownloadBufferFromStreamWithRetry:Stream (stream://AdsBI-AdsMz-Test-MW1-MSNMediation-VE/app/ServiceMaps/MSNMediationServiceMap.ini _ MSNMediation_4a63e32e_91077_2_0-1-28+4a63e32e33) not found in storage ;
Majority of the building errors are temporary and will be fixed automatically. 
If there is no change or progress in the app deployment log after 15 minutes, contact apswat for production environments and aptalk for non-production environment.
Look in App Deployment log for more info

Here are common error messages and possible fixes:
     Msg: Error 53: The network path was not found.: ac: 0
     Fix: Try adding REDMOND@ to the build path.

     Msg: Msg: EDP010385: APSEQREAD::GetDataPointer error 2
     Fix: Try adding REDMOND@ to the build path.

     Msg: Proxy reported error: EDP010196: ApckBuilder Error 3: The system cannot find the path specified.
     Fix: Look for missing directories under your build drop.

For more troubleshooting info, please refer to App Deployment Troubleshooting
```



```
Deployment: 6827694_9979618793807238839_0 (this is NOT the latest deployment.ini change number)
Downloaded [unknown] files, [unknown] bytes.
Failed to import version '6827694_9979618793807238839' of image for environment 'AdsBI-AdsMz-Test-MW1-MSNMediation-VE': Error downloading image from remote storage (stream 'stream://AdsBI-AdsMz-Test-MW1-MSNMediation-VE/app/image.ini@6827694_9979618793807238839')
Majority of the building errors are temporary and will be fixed automatically. 
If there is no change or progress in the app deployment log after 15 minutes, contact apswat for production environments and aptalk for non-production environment.
Look in App Deployment log for more info

Here are common error messages and possible fixes:
     Msg: Error 53: The network path was not found.: ac: 0
     Fix: Try adding REDMOND@ to the build path.

     Msg: Msg: EDP010385: APSEQREAD::GetDataPointer error 2
     Fix: Try adding REDMOND@ to the build path.

     Msg: Proxy reported error: EDP010196: ApckBuilder Error 3: The system cannot find the path specified.
     Fix: Look for missing directories under your build drop.

For more troubleshooting info, please refer to App Deployment Troubleshooting

```

