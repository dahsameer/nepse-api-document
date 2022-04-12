# nepse-api

This document is only for showing how to fetch the data from nepse api. This is only for educational purpose. 

Thanks to [aeruhxi](https://github.com/aeruhxi) for helping me with this.


## First thing first, Getting the access token.

Send a GET request to the api endpoint `https://newweb.nepalstock.com.np/api/authenticate/prove` to get the required details.

The response should look like this.

```json
{
  accessToken: "eyJhbGciOiJIUzI1NiJ9.eyJpc3MiO3iJ5Y28i5LCJzdWIiOiIxMiIsImlhdCI6MTY0OTc2OTc1MSwiZXhwIjoxNjQ5NzY5ODExfQ.e8dEagbDPbwn7QR8nNfMA73Fx5Tn8QyC5KLr_GaRCL0"
  refreshToken: "eyJhbGciOiJIUzI1NiJ9.eyJpc3MiO2iJ5Y283iLCJzdWIiOiIxIiwiaWF0IjoxNjQ5NzY5NzUxLCJleHAiOjE2NDk3NzMzNTF9.WVGqY_LthO_C1BGl5-9GS58cJ2WLSFa8S-U48lfv1mk"
  salt: "Y-w.vG8,I*v5[*-S?i3n"
  salt1: 27431
  salt2: 76353
  salt3: 65229
  salt4: 91802
  salt5: 38487
  serverTime: 1649769751000
  tokenType: ""
}
```

## modifying the token so they are valid

accessToken and refreshToken from this response are not valid. We will need to use the other details provided in the above api to modify our accessToken and refreshToken so that they are valid.

for modifying the accessToken and refreshToken, we will need to implement 2 functions

```javascript
function decode1(saltNum, data){
  return data[((parseInt(saltNum/10) % 10) + (saltNum- (parseInt(saltNum/10) * 10))+(parseInt(saltNum/100) % 10))] + 22;
}

function decode2(saltNum, data){
  return data[(((parseInt(saltNum/10) % 10) + (parseInt(saltNum/100) % 10)) + (saltNum - (parseInt(saltNum/10)*10)))]  + (parseInt(saltNum/10) % 10) + (parseInt(saltNum/100) % 10) + 22;
}
```

which can be called as shown below

```javascript
const dataArr = [9,8,4,1,2,3,2,5,8,7,9,8,0,3,1,2,2,4,3,0,1,9,5,4,6,3,7,2,1,6,9,8,4,1,2,2,3,3,4,4];

//salt1 and salt2 in the arguments of the function call below 
const num1 = decode1(salt2, dataArr);
const num2 = decode2(salt2, dataArr);
const num3 = decode1(salt1, dataArr);
const num4 = decode2(salt1, dataArr);
```

after you get these numbers, we're going to use the numbers to modify our tokens as below.

```javascript
//accessToken and refreshToken are the tokens that you got from the prove endpoint
let validAccessToken = accessToken.slice(0, num1) + accessToken.slice(num1 + 1, num2) + accessToken.slice(num2 + 1);
let validRefreshToken = refreshToken.slice(0, num3) + refreshToken.slice(num3 + 1, num4) + refreshToken.slice(num4 + 1);
```

## getting some random id that is sent in the body for requests

Now, you will need to make a GET request to the endpoint `https://newweb.nepalstock.com.np/api/nots/nepse-data/market-open` with Authorization header as `Salter ${validAccessToken}` as shown below
```javascript
fetch("https://newweb.nepalstock.com.np/api/nots/nepse-data/market-open", {
  "headers": {
    "authorization": `Salter ${validAccessToken}`,
  },
  "body": null,
  "method": "GET",
  "credentials": "include"
})
.then(resp => resp.json())
.then(data => console.log(data));
```
which will give a response like below

```json
{
  "isOpen":"CLOSE",
  "asOf":"2022-04-12T15:00:00",
  "id":80
}
```

As you can see it has a property called id. We will need to use this lets call it `dummyDataId`. To find the id that you need to send to today-price, you will need to do the following calculation.

```javascript
const dummyData = [147, 117, 239, 143, 157, 312, 161, 612, 512, 804, 411, 527, 170, 511, 421, 667, 764, 621, 301, 106, 133, 793, 411, 511, 312, 423, 344, 346, 653, 758, 342, 222, 236, 811, 711, 611, 122, 447, 128, 199, 183, 135, 489, 703, 800, 745, 152, 863, 134, 211, 142, 564, 375, 793, 212, 153, 138, 153, 648, 611, 151, 649, 318, 143, 117, 756, 119, 141, 717, 113, 112, 146, 162, 660, 693, 261, 362, 354, 251, 641, 157, 178, 631, 192, 734, 445, 192, 883, 187, 122, 591, 731, 852, 384, 565, 596, 451, 772, 624, 691];

let currentDate = new Date();
let datePart = currentDate.getDate();
let id = dummyData[dummyDataId] + dummyDataId + 2 * datePart;
```

## calling the today price api

this id is now supposed to use in the body of the request of today-price as shown below

```javascript
fetch("https://newweb.nepalstock.com.np/api/nots/nepse-data/today-price", {
  "headers": {
    "authorization": `Salter ${validAccessToken}`,
    "content-type": "application/json",
  },
  "body": `{"id": ${id}}`,
  "method": "POST",
})
.then(resp => resp.json())
.then(data => console.log(data));
```

which will now let you get the data of today's market.


The arrays that are used above like `dataArr` and `dummyData` are found from studying the source code from wasm file and main javascript file.


This method to get details works currently(as of 2022-04-12) but it might change in the future. I'll also try to make a javascript library in the future since I have not seen any till the moment.
