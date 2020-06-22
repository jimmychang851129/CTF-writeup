# Web Challenges

## XXE attack

### XXE introduction

[xxe attack](https://portswigger.net/web-security/xxe)

#### [sharkctf-2020: XXExternalXX](https://www.tdpain.net/progpilot/sharky2020/)

load a data.xml, enter random strings and inspect the error 

#### [hackpack-2020: XXE attack](https://sec.hamayanhamayan.com/2020/04/30/hackpack-ctf-2020-web-writeups/)

send character '<' and you'll find out things abnormal

## SQL Injection

### ObjectID 

objectID is kind of type storing on Database, in [BSON](https://zh.wikipedia.org/wiki/BSON) datatype

#### [angstromctf-2018](https://www.pwndiary.com/write-ups/angstrom-ctf-2018-the-best-website-write-up-web230/)

kind of sql injection using ObjectID(have to understand the structure of ObjectId) and construct the Object ID

## cookie forge

### jwt cookie forge

#### [hackpack-2020: cookie forge](https://ctftime.org/writeup/20339)

flask web server, cookie in jwt format with

> <base64(json_data)>.<unknown>.<unknown>

get the password(key) in the jwt

> flask-unsign --server https://cookie-forge.cha.hackpack.club/orders --unsign

generate fake cookie

> flask-unsign --sign --secret password1 --cookie "{'flagship': True, 'username': 'tt'}"

## Not yet Categorized

[Aqua world](https://github.com/joshdabosh/writeups/tree/master/2020-SharkyCTF#containment)