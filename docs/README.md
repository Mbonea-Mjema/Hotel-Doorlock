# Introduction

This is a compiled summary of the Rakinda Doorlock API and SDK. The endpoints are defined by the user.

## Contents

<!-- no toc -->

- [Summary](#summary)
- [Events](#events)
- [Java SDK](#sdk)
- [Todo](#todo)

## Summary

> The controller submits a Http request to the server after swiping the card or scan QR Code, and then the server answers, finally the controller operates the action. If offline, the device can deal it by itself. It can validate QR code or IC card or ID card, if it is valid, the door will open. When online, the server will validate the information. When it has no access to internet, offline QR code can also open the door.
> The smartlock

## Events

### `POST` Request Opening

When the guest opens the door the doorlock will send a post requets to the backend server.

`Request Data`

```javascript
{"Type":"00",
"SCode":"1234",
"ReaderNo":"00",
"ActIndex":"00",
"ProjectNo":"00",
"UserName":"admin",
"PassWord":"admin"
}
```

<p style=" text-align: left;">

- **Type**: _upload data type_
  **0** IC card number,
  **1** QRcode/Barcode data;
  **2** ID card number;
  **3** password
  **4** Bluetooth data

- **SCode**: _the data of the upload type above_
- **DeviceID**：_MAC address code of the doorlock_
- **ReaderNo**: _When the device has multiple input peripherals, such as IC reader 1, 2; 2D code reader 1, 2; then 1 means enter and 2 means exit;_
- **ActIndex**: _The relay position of act; it refers to act which reply; generally it is 1, 2; if two relays act at the same time, the value is 3._
- **ProjectNo**: _If not use, this field is null_
- **UserName**: _Some customers need the user name ; if not use, this field is null_
- **PassWord**: _Sometime user password is needed; if not, this field will be null_

</p>

`Response Data`

The response data from the server.

```javascript
{
"ResultCode":"1" // 1 means success; 0 is failure
"ActIndex":"1" // 1 allow to open relay 1, 2 allow to open relay 2,3 to open relay 1 and //relay 2
"Time1":"000A" // set the relay1 delaytime,  Without this field, the default time is taken
"Time2":"0005" // set the relay2 delaytime,  Without this field, the default time is taken
"Audio":"04" // value 04（enter），please check voice list; if not use voice,please don’t //use the“Audio” field
"Msg":"Success" // success or failure validation hint，if device has TFT screen,  The contents //of the MSG field will be displayed on the TFT screen
}
```

### `POST` Upload Data

`Request Data`

```javascript
{"Type":"00",
"SCode":"000000001711010924290100,000000001711010924290101",
"ReaderNo":"00",
"ActIndex":"00",
"ProjectNo":"00",
"UserName":"admin",
"PassWord":"admin"
}
```

<p style=" text-align: left;">

- **SCode**: \_000000001710231214390100
  > 00000000//CardID is 4 byptes
  > 171023121439//yymmddhhmmss
  > 01 //Event number（01 "Remote open door"）
  > 00 //Reader number is 00 \_
- **DeviceID**：_MAC address code of the doorlock_
- **ReaderNo**: _When the device has multiple input peripherals, such as IC reader 1, 2; 2D code reader 1, 2; then 1 means enter and 2 means exit;_
- **ActIndex**: _The relay position of act; it refers to act which reply; generally it is 1, 2; if two relays act at the same time, the value is 3._
- **ProjectNo**: _If not use, this field is null_
- **UserName**: _Some customers need the user name ; if not use, this field is null_
- **PassWord**: _Sometime user password is needed; if not, this field will be null_

</p>

`Response Data`

```javascript
{
"ResultCode":"1",// 1 means success; 0 is failure and device will upload the same event again
"Msg": "success" // success or failure validation hint
}

```

### `POST` Heartbeat

Sends a post request at regular time intervals.

`Request Data`

```javascript
{"DeviceTime":"00",
"Version":"2.34",
"DoorStatus":"01",
"ProjectNo":"00",
"UserName":"admin",
"PassWord":"admin"
}
```

<p style=" text-align: left;">

- **DeviceTime**：_The time of the device_
- **Version**: _Device's software version_
- **DoorStatus**: _00 the first bit is the state of Magnetic switch1，
  the second bit is the state of Magnetic switch 2，
  0 mean the door close,1 mean the door open
  ._
- **ProjectNo**: _If not use, this field is null_
- **UserName**: _Some customers need the user name ; if not use, this field is null_
- **PassWord**: _Sometime user password is needed; if not, this field will be null_

</p>

`Response Data`

```javascript
{
"ResultCode":"1",// 1 means success; 0 is failure
"ActIndex":"1",//if with "ActIndex" field，same as <OPEN API>，can open the relay then //opendoor or gate
"CorrectTime":"211202160630",//with CorrectTime filed，will check device datetime "Msg":"Success"
//if device has tft screen, the Msg content will display on the TFT screen.
}


```

# SDK

The SDK is for remote QR code generation.

```java

import java.text.SimpleDateFormat;
import java.util.Calendar;
import java.util.Date;

public class XXTEACAI {
	public final static int[] KEY = new int[]{//XXTEA encrypt KEY
		0x00543210, 0x00543210,
		0x00543210, 0x00543210
     };
	final static SimpleDateFormat sdfTime = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");


	public static void main(String[] args)
    {
	    String[] strQRArray = {"1","2","3","4","5","6","7","8"};

	strQRArray[0] = "Lily" + "|";//User Definition Data"
	strQRArray[1] = "0000002E";//Close the card number, the user generates a two-dimensional code table, take the ID serial number of this QR code in the table, and the   generated two-dimensional code must make this ID if you want to achieve the limited function is incremental
	strQRArray[3] = "00852953";//User key

	String strTmp1="01";// has a time limit
	String strTmp2="03";// 3 times to open the door
	String strTmp3="02";// Opens the door with a matching of the last 6 bits of the MAC address
	String strTmp4="0000";//default duration
	String strTmp5="000000";//reserved word section
	strQRArray[4] = strTmp1 + strTmp2 + strTmp3 + strTmp4 + strTmp5;//admin byss

	//Get the start time and end time, in this case the start time is the current time, the end time is the current time + the time after three days
	//The start time is 5 minutes ahead of the current time, which eliminates the problem of 1 minute
	    Date date=new Date();
	    SimpleDateFormat df = new SimpleDateFormat("yyyyMMddHHmmss");//Format dates
	strTmp1 = df.format(date.getTime()-600000);//The current time minus 10 minutes, lest there be no effective time
	    strTmp2 = strTmp1.substring(2, strTmp1.length());

	    Calendar calendar1 = Calendar.getInstance();
	    df = new SimpleDateFormat("yyyyMMddHHmmss");//Format dates
	    calendar1.add(Calendar.DATE, 3);
	    strTmp3 = df.format(calendar1.getTime());
	    strTmp4 = strTmp3.substring(2, strTmp3.length());

	    strQRArray[5] = strTmp2 + strTmp4;

	strTmp1 = "03";//Two matching MAC addresses
	strTmp2 = "112ED3"; // 2 MAC addresses followed by 6 bits
	    strTmp3 = "920993";
	    strTmp4 = "920994";
	    strQRArray[6] = strTmp1 + strTmp2 + strTmp3 + strTmp4;

	//A string that needs to be encrypted
	    String encryptCodeStr = strQRArray[4] + strQRArray[5] + strQRArray[6];

	    //encryptCodeStr="010302000000000017041116080717041416080702920992";
//	The encryption length is 2 bits
	    int   encryptCodeLength = encryptCodeStr.length();
	byte encryptLength = (byte)(encryptCodeLength/8);//XXTEA Number of encrypted shapes = encrypted string length/8; +1 if the remainder is not 0 if the remainder is not 0
	    byte nRemainder1 = (byte) (encryptCodeLength%8);
	    if(nRemainder1!=0)
	    {
		    encryptLength++;
	    }

        int v = ((int)encryptLength) & 0xFF;
        strTmp1 = Integer.toHexString(v);
        if (strTmp1.length() < 2) {
	        strTmp1 ="0"+strTmp1;
        }

	// Get the remainder of the encrypted string Half of the  remaining 8 of the encrypted string
        byte nRemainder = (byte) ((encryptCodeLength%8)/2);

        v = ((int)nRemainder) & 0xFF;
        strTmp2 = Integer.toHexString(v);
        if (strTmp2.length() < 2) {
	        strTmp2 ="0"+strTmp2;
        }
	// Get the encryption length and remainder
	    strQRArray[2] = strTmp1 + strTmp2;
	// Converts a string to a decimal integer
	// Turn the first 8 integers first, and then the last integer
	    int[] EnCodeInt = new int[30];

        int stmp1 = 0;
        int stmp2 = 0;
	    if(nRemainder!=0)
	    {
		// The string is divided into one integer value, and because the range of JAVA Integer.valueOf() integer values is limited, the integer value is divided into two 4-character values
		    for(int k=0;k<encryptLength-1;k++)
		    {
			    strTmp4 = encryptCodeStr.substring(k*8,k*8+4);
			    strTmp5 = encryptCodeStr.substring(k*8+4,(k+1)*8);
	            stmp1 = Integer.valueOf(strTmp4,16);     //00852953
	            stmp2 = Integer.valueOf(strTmp5,16);
	            EnCodeInt[k] = (stmp1 << 16) | stmp2;
			    //EnCodeInt[k] = Integer.valueOf(encryptCodeStr.substring(k*8,(k+1)*8),16);
		    }
		       EnCodeInt[encryptLength-1] = Integer.valueOf(encryptCodeStr.substring((encryptLength-1)*8,(encryptLength-1)*8+nRemainder1),16);
	    }else{
		    for(int k=0;k<encryptLength;k++)
		    {
			    strTmp4 = encryptCodeStr.substring(k*8,k*8+4);
			    strTmp5 = encryptCodeStr.substring(k*8+4,(k+1)*8);
	            stmp1 = Integer.valueOf(strTmp4,16);     //00852953
	            stmp2 = Integer.valueOf(strTmp5,16);
	            EnCodeInt[k] = (stmp1 << 16) | stmp2;
			    //EnCodeInt[k] = Integer.valueOf(encryptCodeStr.substring(k*8,(k+1)*8),16);
		    }
	    }
	// Set the user key, the device key is fixed to 0x00543210, if you want to modify the device key, you need to set the device key for the device

        stmp1 = Integer.valueOf("0085",16);     //00852953
        stmp2 = Integer.valueOf("2953",16);
        KEY[0] = (stmp1 << 16) | stmp2;
        KEY[2]  = KEY[0];

	    int[] EnCodeResult = new int[20];
	    EnCodeResult = encrypt(EnCodeInt, KEY,encryptLength);
	    String EnQRResult="";
	    for(int k=0;k<encryptLength;k++)
	    {
		    strTmp1 = pad(Integer.toHexString(EnCodeResult[k]).toUpperCase(), 8, true);
		    EnQRResult += strTmp1;
	    }
	    System.out.println("pre-encryption string" + encryptCodeStr);
	    System.out.println("encrypted string" + EnQRResult);

	// Test to decode the encrypted string
	    int[] Decodeint = new int[20];
	    Decodeint = decrypt(EnCodeResult, KEY,encryptLength);
	    String DeQRResult="";


	// After decoding the string, first solve the front, and finally solve the last integer value, the last integer number

	// If the number of digits is not an integer of 8 and there is a remainder, the excess 00 in front should be removed according to the QR code remainder parameter

	    for(int k=0;k<encryptLength-1;k++)
	    {
		    strTmp1 = pad(Integer.toHexString(Decodeint[k]).toUpperCase(), 8, true);
		    DeQRResult += strTmp1;
	    }
	if(nRemainder==1)// The remainder is 1, which represents the last string, which has 2 bits
	    {
		    strTmp1 = pad(Integer.toHexString(Decodeint[encryptLength-1]).toUpperCase(), 2, true);
		    DeQRResult += strTmp1;
	    }
	else if(nRemainder==2)// The remainder is 2, representing the last string, which has 4 bits
	    {
		    strTmp1 = pad(Integer.toHexString(Decodeint[encryptLength-1]).toUpperCase(), 4, true);
		    DeQRResult += strTmp1;
	    }
	else if(nRemainder==3)// The remainder is 3, which represents the last string, which has 6 bits
	    {
		    strTmp1 = pad(Integer.toHexString(Decodeint[encryptLength-1]).toUpperCase(), 6, true);
		    DeQRResult += strTmp1;
	    }
	else // The remainder is 0, indicating that the last string, which has 8 bits, is complete
	    {
		    strTmp1 = pad(Integer.toHexString(Decodeint[encryptLength-1]).toUpperCase(), 8, true);
		    DeQRResult += strTmp1;
	    }
	    System.out.println("decrypted string" + DeQRResult);

	    String QREndStr = strQRArray[0] + strQRArray[1] + strQRArray[2] + strQRArray[3] + EnQRResult;
	System.out.println("Full 2D code string:"+QREndStr);



}


	private static String pad(String str, int size, boolean isprefixed) {
		if (str == null)
			str = "";
		int str_size = str.length();
		int pad_len = size - str_size;
		StringBuffer retvalue = new StringBuffer();
		for (int i = 0; i < pad_len; i++) {
			retvalue.append("0");
		}
		if (isprefixed)
			return retvalue.append(str).toString();
		return retvalue.insert(0, str).toString();
	}

	private static java.lang.String toHexString(int encryptLength) {
		// TODO Auto-generated method stub
		return null;
	}

	public static int[] encrypt(int[] v, int[] k,int n) {
        int and;
        int p;
        int rounds = 6 + 52/n;
        int sum = 0;
        int z = v[n-1];
        int delta = 0x9E3779B9;
        of {
            sum += delta;
            int e = (sum >>> 2) & 3;
            for (p=0; p<n-1; p++) {
              y = v[p+1];
              z = v[p] += (z >>> 5 ^ y << 2) + (y >>> 3 ^ z << 4) ^ (sum ^ y) + (k[p & 3 ^ e] ^ z);
            }
            y = v[0];
            z = v[n-1] += (z >>> 5 ^ y << 2) + (y >>> 3 ^ z << 4) ^ (sum ^ y) + (k[p & 3 ^ e] ^ z);
          } while (--rounds > 0);

        return v;

    }

    public static int[] decrypt(int[] v, int[] k,int n) {
        //int n = v.length;
        int z = v[n - 1], y = v[0], delta = 0x9E3779B9, sum, e;
        int p;
        int rounds = 6 + 52/n;
        sum = rounds*delta;
        y = v[0];
        of {
          e = (sum >>> 2) & 3;
          for (p=n-1; p>0; p--) {
            z = v[p-1];
            y = v[p] -= (z >>> 5 ^ y << 2) + (y >>> 3 ^ z << 4) ^ (sum ^ y) + (k[p & 3 ^ e] ^ z);
          }
          z = v[n-1];
          y = v[0] -= (z >>> 5 ^ y << 2) + (y >>> 3 ^ z << 4) ^ (sum ^ y) + (k[p & 3 ^ e] ^ z);
        } while ((sum -= delta) != 0);
        return v;
    }

}



```

## Todo List

- [x] Translation
- [] Remote Unlocking
