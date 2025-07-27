# Cookies

## Import cookies

We can use the developer tools of our browser.

We can use [cookie editor](https://cookie-editor.com/) plugin.

### Import cookies with cuddlephish

We can use [cuddlephish](https://github.com/fkasler/cuddlephish) stealer.js script. However this requires
the cuddlephish format and it import cookies in a new browser.

Here is an example:
```
git clone https://github.com/fkasler/cuddlephish
cd cuddlephish
npm install
node stealer.js cookies.json
```


Here is an example of the supported format of cuddlephish:
```json
{
    "url": "https://myaccount.google.com/?pli=1",
    "cookies": [
        {
            "name": "OTZ",
            "value": "8170000_56_56__56_",
            "domain": "accounts.google.com",
            "path": "/",
            "expires": 1755095978,
            "size": 21,
            "httpOnly": false,
            "secure": true,
            "session": false,
            "priority": "Medium",
            "sameParty": false,
            "sourceScheme": "Secure",
            "sourcePort": 443
        }
    ],
    "local_storage": [
    ]
}
```


## Google cookies

- [Mastering Web Cookies](https://medium.com/jungletronics/mastering-web-cookies-e88388aee5ad)


