# Chrome

## Chrome cookies

Chrome stores its cookies inside a SQLite database located in:

Windows location:
```
C:\Users\<username>\AppData\Local\Google\Chrome\User Data\Default\Network\Cookies
```

GNU\Linux location:
```
/home/<username>/.config/google-chrome/Default/Cookies
```

- [Where does Chrome store cookies?](https://stackoverflow.com/questions/31021764/where-does-chrome-store-cookies)


However, the Chrome cookies are encrypted in the Cookies database,
protected by App-Bound-Encryption (ABE) that only allows the Chrome
process to decrypt them. More info in [Chrome App-Bound Encryption (ABE) - Technical Deep Dive & Research Notes](https://github.com/xaitax/Chrome-App-Bound-Encryption-Decryption/blob/main/docs/RESEARCH.md).


Here are some tools that allows to extract
the cookies from Chrome in Windows:

- [Chrome-App-Bound-Encryption-Decryption](https://github.com/xaitax/Chrome-App-Bound-Encryption-Decryption) by xaitax
- [Cookie-Monster-BOF](https://github.com/KingOfTheNOPs/cookie-monster) by KingOfTheNOPs


## Chrome history

The chrome history is stored in a plain SQLite database located in the
following places:

Windows Location:
```
C:\Users\<username>\AppData\Local\Google\Chrome\User Data\Default\History
```

GNU\Linux location:
```
/home/<username>/.config/google-chrome/Default/History
```

## Injection into Chrome

Related to DLL hijacking, Chrome performs a hash check for the system DLLs it
loads. Chrome returns the error `STATUS_INVALID_IMAGE_HASH`.

It has a check to disable the [Disable RendererCodeIntegrity](https://www.technipages.com/fix-google-chrome-status_invalid_image_hash-error/#1_Disable_RendererCodeIntegrity)
value, but it seems that works only for the render.


## Resources

- [HackBrowserData](https://github.com/moonD4rk/HackBrowserData) by moonD4rk
