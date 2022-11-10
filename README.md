# PDF 2.0 Extensions proof of concepts

This repository contains proof of concept implementation of following ISO documents:
* ISO/WD TS 32003 Document management — Portable Document Format — Adding support of AES - GCM in PDF 2.0 
* ISO/WD TS 32004 Document management — Portable Document Format — Integrity protection in encrypted document in PDF 2.0 

## Adding support of AES - GCM 
Advanced Encryption Standard with Galois Counter Mode (AES-GCM) is a block cipher mode of AES algorithm that provides authenticated encryption. The extension introduces new version in Encrypt dictionary and revision in standard security handler.  
Provided implementation is enhancing password-based security handler with the AES-GCM to encrypt and decrypt input files according ISO/WD TS 32003.  
When decrypting, the implementation raises standard errors (wrong password provided) and an error when the GCM authentification fails (Auth Tag check fails).  
AES-GCM is known for weakness when reusing initialization vector(IV). Our implementation has an option to enforce the use of same IV for study purpose.  

## Integrity protection in encrypted document
Combination of encrypted file, incremental update and /Identity crypt filter may result in wrong impression of a recipient that the protected file can't be changed. ISO/WD TS 32004 offers provisions that protects integrity of the content through Message Authentification Code (MAC) mechanism. The proposal in theory works on any standard security handler because the functionality is introduced through #13 bit in permissions. Our implementation offers combination of AES-GCM password securoty and integrity protection only.  

# Content
PoC implementation is done using Foxit PDF SDK for Windows and can be found in /bin folder.  
Sample pdf filed files are in /pdf folder.  


## Usage

Usage: encrypt-poc.exe **[options]** `<argument>`  
options:  
 **-e** without argument. encrypts the `<input>` file and saves in the `<output>` file.  
 **-d** without argument, decrypts the `<input>` document and saves in the `<output>` file.  
 **-dr** without argument, decrypts the `<input>` file and renders pages in the `<output>` folder.  
 **-op** `<owner_password>` provides owner password.  
 **-up** `<user_password>` provides user password.  
 **-p** `<permissions>` provides permissions. If bit #13 is NOT set, 	then Integrity protection (ISO 32004) is applied. default=-4 all allowed, no integrity protection.  
 **-m** without argument, if provided, EncryptMetadata will be set to true.   
 **-iv** without argument, if provided the IV (initialization vector) is the same for every object.  
 **-i** `<input>`, input file.  
 **-o** `<output>`, output file or folder.  

### Usage samples
```
  encrypt-poc.exe -e -i "c:\Hello world.pdf" -o "c:\Hello world encrypted.pdf" -op owner -up user -p -4
  
Creates new file with AES-GCM encryption, default permission (everything is allowed, no integrity protection s per ISO 32004) owner password is "owner" and user password is set to "user"  
```

```
  encrypt-poc.exe -e -i "c:\Hello world.pdf" -o "c:\Hello world encrypted with IP.pdf" -op owner -up user -p -4396  

Creates new file with AES-GCM encryption. Owner password "owner" and user password is set to "user" The integrity protection as per ISO 32004 is set (bit #13 is not set) and bits #4 and #6 are not set, that means modifying the contents and annotations are dissalowed  
```

```
  encrypt-poc.exe -d -i "c:\Hello world encrypted with IP.pdf" -o "c:\Hello world decrypted.pdf" -op owner 
  
Opens encrypted file with provided "op" password and saves decrypted file  
```

## Sample files
all files are protected with password variation of standard security handler with AES-GCM encryption algorithm.  
Owner password used: "owner"  
User password used: "user"  

File | Algorithm | Permissions | Additional settings |
--- | --- | --- | --- |
1.pdf | AES-GCM | everything is allowed | metadata unencrypted 
2.pdf | AES-GCM | everything is allowed | metadata encrypted
3.pdf | AES-GCM | everything is allowed | all initialization vectors are intentionaly zeroes
4.pdf | AES-GCM | everything is allowed | all AuthTags are intentionaly malformed
5.pdf | AES-GCM | content and annotations modifying is dissallowed | Integrity protection is enabled, metadata unencrypted
6.pdf | AES-GCM | content and annotations modifying is dissallowed | Integrity protection is enabled but the MAC is intentionaly malformed


