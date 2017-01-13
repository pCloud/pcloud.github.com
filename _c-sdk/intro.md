---
layout: doc
title:  "pCloud Linux C SDK"
categories: sdk dll c
---

## Usage
You can use pre-build libraries or build them your self for your system. 
There are build and packeges for Windows and Linux

### Binary packages
- Binary packages Linux and Winodws
  [Prebuild shared libs for Linux and Windows](https://my.pcloud.com/publink/show?code=kZdfvxZEPE5fXMCLDHCmo4CaSGQEHPqq3uV)   

### Required libraries 
- Both Windows and Linux
[Zlib](http://zlib.net/)  A Massively Spiffy Yet Delicately Unobtrusive Compression Library.  
[mbedtls](https://tls.mbed.org/) mbed TLS (formerly known as PolarSSL) makes it trivially easy for developers to include cryptographic and SSL/TLS capabilities in their (embedded) products, facilitating this functionality with a minimal coding footprint.  
[Pthread](http://www.gnu.org/)   

On Ubuntu you can run the following command:  
> sudo apt-get install zlib1g-dev  libpthread-stubs0-dev 

## Documentation 
```
  /*
   * Initialization function. First to be called in order to initialize the library. Upon failure int error is returned 
   *and err parameter gets text message of the error. The err needs to be freed if there was an error. The Possible faults are:
   *1 - no memory to initialize the library, 2 - Failed to initialize SSL library, 3 - failed to initialize pthered mutex. 
   *0 - success. The save_auth parameter indicates weather authentication information is to be saved in a file between runs of 
   *the library.
   */
```
- int psdk_init(int save_auth, char **err);

```
  /*
   * Allows login for users of external application that has corresponding reference id. Only paid customers registered from 
   *this application can be logging this way.
   */
```
- int psdk_set_user_pass(const char * user, const char * pass, int save_auth, uint64_t ref);

```
  /*
   * The method implements OAuth 2.0 authorization process that allows external applications to obtain access to users account
   *with user's agreement. First call psdk_authorize to obtain URL where client can authorize the access to hist account. Open 
   *the obtained URL in browser or print it for the client to paste it himself/herself. Then if wait parameter is 1 all calls will 
   *block until clients authorization and if wait is 0 you can call psdk_wait_authorized to block until authorization acquired.
   *Bare in mind that wait 1 with psdk_wait_authorized will most probably deadlock. The clientid and clientsicret parameters 
   *can be obtained from pcloud application management console: https://docs.pcloud.com/oauth/index.html
   */
```

-  char * psdk_authorize(const char *clientid,const char *requestid, char * clientsicret, int wait);

-  void psdk_wait_authorized() ;

```
  /*
   * Creates an upload file task and reports completion progress in the given callback. 
   */
```

-  int psdk_upload_file(const char* localpath, const char *filename, const char * remotepath, CompletitionCallback callback);

```
  /*
   * Creates a download file task and reports completion progress in the given callback. 
   */
```

-  int psdk_download_file(const char* localpath, const char *filename, const char * remotepath, CompletitionCallback callback);

```
  /*
   * For all calls that take char **err and return int success is 0 and an error code is returned otherwise and err is populated with 
   *string description of the problem. In case of error err MUST BE FREED after it is used.  
   */
```

-  int psdk_rename_file(const char *path, const char *topath,  char **err);

-  int psdk_delete_file(const char *path, char **err);

```
  /*
   * Checks and creates new folder with write permissions on it and adds suffix to the name if necessary i.e. New Folder (1) etc..
   */
```

-  int psdk_check_create_folder(const char * path);

```
  // Creates remote folder no checks no suffix
```

-  int psdk_create_folder(const char *path, char **err);

-  int psdk_delete_folder(const char *path, char **err);

-  int psdk_rename_folder(const char *path, const char *topath,  char **err);

```
  /*
   * Returns JSON representation of the description of the contents of given remote folder. On err NULL pointer is returned and 
   *err is filled up with string representation of the error in JSON form. If err is filled it must be freed. 
   */
```

-  char * psdk_list_folder(const char *path,  char **err);

```
  /*
   *General purpose send API call function that can be used to issue different API commands and retrieve result as JSON. 
   *Parameter command is the name of the command, list of available commands and their respective responses can be found 
   *here https://docs.pcloud.com/.
   *Parameter result is populated with JSON representation of the response, needs to be freed. 
   *If the method requires login, login parameter should be set to 1, otherwise to 0.
   *The numparam is number of parameters that are going to be passed.
   *The fmt is format string with the type of the parameters for where %s is string %n is numeric and %b is boolean. For example we pass 
   *3 parameters  path, fileid,expire of types string int and bool, the corresponding function call would be:
   *int res = psdk_send_api_command("command_name", result_json, 1, 3, "path%s fileid%n expire%b", path, fileid, 1);
   */
```

-  int psdk_send_api_command(const char *command, char **result, int login, int numparam, const char * fmt, ...);

-  void psdk_stop();

## Example 

```
#include "pSDK.h"
#include 
#include 
#include 

int main()
{
  char *err = NULL;
  psdk_init(1, &err);
  
  char * authurl = psdk_authorize("lCmd5dtXNYX","1034", "LKAqbD8y39L93Y3WtdbRHy2mLQ8y", 1);
  printf ("Paste this in browser [%s]\n", authurl );
  
  //psdk_wait_authorized();

  char * res = psdk_list_folder("/", &err);
  
  printf ("List root: \n%s\n", res);
  
  free (authurl);
  free(res);
  free(err);
  
}
```