Jun 2026 downgrade to 1.0.7 success


what you need: 

1. charles(mine is latest V5.2)
2. earbuds, better verify the battery status to decide firmware replacement or battery replacement. For red/white flashing issue, using jbruneaux31's quick fix at https://github.com/kioipp/bose_earbuds_firmware/issues/4 is a better choice, as a quick verification. For me, I have a pair of working 2.0.7 earbuds, and before this downgrade, I used above fix and refreshed the firmware, though still 2.0.7 with that fix.
3. the bin file (lando_2x1_rom_spin_encrypted_prod_1.0.7-620b71c.bin) in this repo, actually this is the earbuds firmware. Download it to local computer, then, either upload it to AWS s3, Azure file or aliyun OSS(kind of static file host service). You can also use hfs to host it locally but I didn't succeed on that. Expected result is that you can download the bin file using an url in the browser. For example, I can download the file after putting this(https://XXXXXXXXXXXXXXXXXXX.oss-cn-shanghai.aliyuncs.com/123.bin) in my browser. By doing this, we can let the bose updater to download the firmware from desired url but not official bose cdn site anymore

steps:

first step is same with jbruneaux31's quick fix at https://github.com/kioipp/bose_earbuds_firmware/issues/4

1. install Charles and configure it to support SSL (follow guide on the Charles website for general SSL procedure, including installing certificate as the procedure has slightly changed, then add the following urls to the SSL Proxying settings. Menu Proxy->SSL Proxying Settings. In tab SSL Proxying, use the 'Add' button):

updates-framingham-prod.smartproducts.bose.io
btu.bose.com (although this one is not required, it is still interesting to see the content)


Below steps are different from jbruneaux31's quick fix (due to the reason that my earbuds are downgraded already, I can't make the complate screenshots , some of which are only available when earbuds are 2.0.7)

2. we enable charles endpoints first, menu-proxy-enable breakpoints(Ctrl+K)
![2_enable_breakpoints](https://github.com/nasmatic/bose_qc_earbuds_downgrade_1.0.7/blob/main/2_enable_breakpoints.png)


3. In Charles-proxy-breakpoints settings, add one rule, leave Method empty(I guess setting it to GET also works),Protocol: https, Host:updates-framingham-prod.smartproducts.bose.io,Port:443,Path:/update,Query:*(wildcard), then click done. When bose updater requests updates-framingham-prod.smartproducts.bose.io, it will be intecepted.
![3_breakpoints_setting](https://github.com/nasmatic/bose_qc_earbuds_downgrade_1.0.7/blob/main/3_breakpoints_setting.png)

4. Goto btu.bose.com and let is go up to the point where it says that the firmware is already up-to-date.If you haven't installed the Bose updater, then download and install and run. This software is like a background process briging your earbuds and web based interface.
![4_request_modifying](https://github.com/nasmatic/bose_qc_earbuds_downgrade_1.0.7/blob/main/4_request_modifying.png)

5. If everything is ok, when web interface changes serveral times, in charles, there will another tab called breakpoints. In this tab, request is intecepted and is waiting for user's action. We change productType to lando(which is QC earbuds's alias), and we change firmwareVersion from 2.0.7 to 1.0.7-9846+620b71c. Finally we click execute.By doing this, we tell bose updater this earbuds has a 1.0.7 firmware and needs update.
6. Next, still in Charles, another breakpoints is waiting for user's action. This time we need to edit the reponse. Orignial response should be like this:

{
	"updateAvailable": true,
	"baseUrl": "https://ota.cdn.bose.io/c60fd5e1-1e8e-4df9-ab1a-ed8723397150.bin/c60fd5e1-1e8e-4df9-ab1a-ed8723397150.bin",
	"versionInfo": "2.0.7-18347+fb87694",
	"deferrable": false,
	"metadata": {
		"CRC": "0X2D839DDC",
		"length": 6534306,
		"estimatedInstallDuration": 0,
		"speedTestLink": ""
	}
}

This is Bose's offical firmware download endpoint. Here, we replace baseUrl to your own file host endpoint(AWS, AZure, Aliyun), Make versionInfo unchanged. Change length to 6341544, change crc to 0x7CF57460. Then execute. By doing this, we guide bose updater to download the firmware from your own file endpoint. If you see empty reponse after modifying the request, just execute and another request will be intercepted, and you'll see the normal response from Bose official.

7. If everything works ok, you should see the green update button on webpage. Click and wait for process. 

8. Error will happen in the end of the updating. Ignore it and unplug the earbuds cable. Re-pairing it to your phone and check Bose app if the firmware has been downgraded to 1.0.7. For me, after downgrading, the interface of Bose app also changes, where you can swipe to feel different NC levels.
![8_after_downgrading_1](https://github.com/nasmatic/bose_qc_earbuds_downgrade_1.0.7/blob/main/8_after_downgrading_1.png)
![8_after_downgrading_2](https://github.com/nasmatic/bose_qc_earbuds_downgrade_1.0.7/blob/main/8_after_downgrading_2.png)

Also a chinsese tieba thread for reference:https://tieba.baidu.com/p/9184928301?fr=undefined
