# FirstRepo
This is our first repository


var str = new String("Dhandapani Sudhakar");
for(var i=0; i<str.length; i++) {
    console.log(str.charAt(i));
}

var lengthOfStr = str.length;
console.log(lengthOfStr);

var str1 = new String("ABCDEFGH");
for(var i=0; i<str1.length; i++) {
    console.log(str1.charCodeAt(i));
}

var str2: string = "Kathirvel";
var str3: string = " Sudhakar";
var str4 = str2.concat(str3);
console.log(str4);

var str5 = "I am Dhandapani Sudhakar";
var index1 = str5.indexOf("S");
var index2 = str5.indexOf("s");
var index3 = str5.indexOf("k");
var index4 = str5.lastIndexOf("a");
console.log(index1);
console.log(index2);
console.log(index3);
console.log(index4);

var str6 = "I am Dhandapani Sudhakar";

var val1 = str6.localeCompare("I am Dhandapani Sudhakar");
var val2 = str6.localeCompare("Sudhakar Dhandapani am I");
var val3 = str6.localeCompare("I am Kathirvel Sudhakar");
var val4 = str6.localeCompare("I am Dhandapani Arunthathi");

console.log(val1);
console.log(val2);
console.log(val3);
console.log(val4);

var str7 = "Everything is good";
var regEx = /good/;
if(str7.search(regEx)==-1){
    console.log("Does not contain good")
}
else {
    console.log("Contains good");
}

var str8 = "Ennampol Vazhkai";
var regEx1 = /Vazhkai/;
var str9 = str8.replace(regEx1, "Vazhvu");
console.log(str9);

var str10 = "Bharathiyar is one of the ancient Tamizh poet";
var slicedStr1 = str10.slice(0,);
var slicedStr2 = str10.slice(0,12);
console.log(slicedStr1);
console.log(slicedStr2);

var str11 = "Oh God Beautiful poem is written by Guru Nanak";
var splittedArray1 = str11.split(" ",5);
var splittedArray2 = str11.split("",5);
var splittedArray3 = str11.split("",);
console.log(splittedArray1);
console.log(splittedArray2);
console.log(splittedArray3);

var str12 = "Ozymandiaz is my Hero";
console.log(str12.substr(0,12));
console.log(str12.substr(-1,2));
console.log(str12.substr(12,25));

var str13 = "Tere Mere Beach Mae";
console.log(str13.toString());
console.log(str13.valueOf());
console.log(typeof str13);

var str14 = new String("TypeScript");
console.log(str14 instanceof String);





