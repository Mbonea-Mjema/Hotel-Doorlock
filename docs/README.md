# Introduction

This is a compiled summary of the Rakinda Doorlock API and SDK.

## Contents

<!-- no toc -->

- [Summary](#summary)
- [Events](#events)
- [Java SDK](#sdk)

## Summary

> The controller submits a Http request to the server after swiping the card or scan QR Code, and then the server answers, finally the controller operates the action. If offline, the device can deal it by itself. It can validate QR code or IC card or ID card, if it is valid, the door will open. When online, the server will validate the information. When it has no access to internet, offline QR code can also open the door. Surely the QR code should comply with the principles of opening the door. After connecting to the internet, the records of opening the door will be uploaded automatically. If the times of opening the door should be recorded, this information will also be uploaded after connecting to the internet. This is to say if some QR code can only open the door once, after online opening, when in offline condition later, this QR code could not open the door anymore.

## Events

### `POST` Request Opening

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
- **DeviceID**：_MAC address code of the device, 12 length_
- **ReaderNo**: _When the device has multiple input peripherals, such as IC reader 1, 2; 2D code reader 1, 2; then 1 means enter and 2 means exit;_
- **ActIndex**: _The relay position of act; it refers to act which reply; generally it is 1, 2; if two relays act at the same time, the value is 3._
- **ProjectNo**: _If not use, this field is null_
- **UserName**: _Some customers need the user name ; if not use, this field is null_
- **PassWord**: _Sometime user password is needed; if not, this field will be null_

</p>

`Response Data`

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
- **DeviceID**：_MAC address code of the device, 12 length_
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

## How it works

Two-dimensional code rules are composed of seven parts, the first four parts are plaintext, the last three parts of the string through the XXTEA encryption

1. User-defined Data，n bits
   User-defined data that does not contain the symbol "|", because it is an identifier that separates user data from other data. User data refers to additional data added by users, such as name, position, etc

2. Two dimensional code number（8bits）
   After the two-dimensional code has opened the door, a record will be generated, Record contains the two dimensional code number number. if enable the timers limit , and the two dimensional code number is incremented by 1 ,and ensure that the two dimensional code number is incremented without repetition
3. Encryption length (2 bits) and encryption length remainder(2bits）
   Note:

1) Encryption length, refers to the encryption string every 8 bits, then the encryption length is 1 ,16 bits, then the encryption length is 2 , 18 bits, then the encryption length is 3  
   2)The remainder of the encryption length is last 2-6 bits character.
   If the remaining 2 bits, the remainder is ‘01’, If the remaining 4 bits, the remainder is ‘02’, If the remaining 6 bits, the remainder is ‘03’, If there is no residue, the remainder is ‘00’.

4. user secret key（UserKey2），4bytes
   From Sector 5-7 are encrypted using the XXTEA encryption method. The length of the XXTEA encryption key is 256 bits (32 bytes).The encryption key consists of the device key (UserKey1) and the user key (UserKey2). The device key (UserKey1) defaults to 0x00543210, and the user key (UserKey2) is defined by user-define. unsigned int key[4] = { UserKey2, UserKey1, UserKey2, UserKey1};
5. Management part 16bits
   Time limit 2bits
   00 no time limit.
   01 Start time and end time limit  
   Timers 2bits Range of value :00-FF
   00 no timer limited.
   01-FF ( 1-255)
   Open mode 2bits
   00 direct Open
   01 Project No. +Room No. match to open the door.
   02 The last 6 bits of MAC address match to open the door
   03 The last 4 bits of MAC address match to open the door
   Opening delay 4bits （FFFF: Max 65535 seconds）
   0000 default
   0012 18seconds
   Opening delay: the relay controls the time of power failure or power on.

Expanding second output 1bit
0 No expansion, no second output, compatible with previous format
1 the output of second channels (when the two dimensional code is used, the second output will
act simultaneously with the first door lock)
Extended Wigan output 1bit
0 No extension, no output of Weigan output, compatible with previous format
1 Output of Weigan output

4 bits reserved for extension purposes

6. the second output. 4bits
   If the second output is enabled, the second output relay output delay time is 8 bits (if the extended second output is disable, this field does not exist).
   0000 default
   003C 60 second
   Reserved 4 bits for extension

7. Weigan number. 8bits
   if you extend the output of Weigan output, Weigan number is 8 bits.
   If the output of Weigan card is weigen 26, The highest two bits of the card series number is ‘00’

8. the effective date of opening the door: the start date and the end date 24 bits
   (if no time limit. Fill in 24 characters at will)
   for example:160227101256160228101256  
   The starting time of this two-dimensional code : 2016-02-27 10:12:56
   The ending time of this two-dimensional code : 2016-02-28 10:12:56

8．Combination of 6 rooms or MAC addresses
There are two ways to open and match authentication:1) MAC，2)Project No. and Room No.，According to the situation, the two choose one.
If the room authentication matching method is selected, the Room number of the equipment needs to be pre-set to the equipment.

Project and room authentication method: 1) Number of rooms： 2bits
02 There are 2 rooms
2)Project No: 8bits
00000000 default
41590501 ProjectNo. 3) Room number combination 8bits\*N  
 0302010103020102

MAC authentication method: 1) Number of MACs： 2bits
02 There are 2 devices’ MAC 2) MACs combination 6bits*N / 4bits*N
030201030202

```Java

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
