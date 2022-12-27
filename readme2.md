# auto-login wifi.id dan wms 2022
#
```code
:local currentQueue 0;
:local nolock true;
:local gurl "https://welcome2.wifi.id";
:local guxx "http://www.msftconnecttest.com/redirect";
:local username "";
:local password "";
:local int "ether1";
:local gwp "WAG-DX-PTR";
:local mdlan "MSTMA000xx-N/TLK-CI-xx33x:xxx";
:local type "WMS";

:while (true) do={ 
 :do {
  :delay 10;
  :if ([/ping address=8.8.8.8 count=1] = 0) do={
   :set currentQueue ($currentQueue +1);
   
   :if ($currentQueue > 3) do={
    :if (nolock) do={	
	 :set nolock false;
	 
     :global gip [/ip address get [/ip address find interface=$int] address];
     :global gip [put [:pick $gip 0 [:find $gip "/"]]];
     :global gmc  [/interface wireless get $int mac-address];   
     :global vrab "ipc=$gip&gw_id=$gwp&mac=$gmc&redirect=$guxx&wlan=$mdlan";
     :global url "$gurl/authnew/login/check_login.php\?$vrab";
     :global post "username=$username@spin2&password=$password";
   
     :if ($type = "WMS") do={
      :set username "xxx"
      :set password "xxx"
      :set url "$gurl/wms/auth/authnew/autologin/quarantine.php\?$vrab"
      :set post "username_=$username&username=$username.uZLs%40freeMS&password=$password"
     };
	
     log warning ("Internet Mati Memulai Koneksi Ulang | $gip | $gmc | $type | $currentQueue"); 	
     :set currentQueue 0;
	
	 :do {
	  /interface disable $int
      :delay 5;
      /interface enable $int
	  :delay 10
	 } on-error={
      log warning ("Error set");
     };

     :do {
	  :local result [/tool fetch http-method=post http-data=$post url=$url http-header-field="User-Agent: Indihomo" as-value output=user];
      :if ([:find ($result->"data") "Sukses"] >= 0) do={
       log info ("Internet Kembali Normal")
      } else={
       :if (($result->"data") = "") do={
        log info ("Internet sudah konek");
       } else={
        log warning ($result->"data")
       };
      };
	 } on-error={
      log warning ("Error Cek Internet");
     };
	 
	 :set nolock true;
	 
	} else={
	 log warning ("Lock Proses");	 
	};
	
   };
   
  } else {
   # log info ("Internet Normal");
   :set currentQueue 0;
  };
  
  } on-error={
   log warning ("Error big");
   :set currentQueue ($currentQueue +1);
  };
}
```
