
# Table of Contents

1.  [SSH2 SFTP Client](#org2ff92dd)
2.  [Installation](#org101da24)
3.  [Basic Usage](#orge0baaf0)
4.  [Breaking Changes in Version 4.x](#org1450b73)
5.  [Enhancements in Version 4.2.x](#org0da6002)
6.  [Enhancements in Version 4.1.x](#org436dffc)
7.  [Documentation](#org627f927)
    1.  [Methods](#org2e64c16)
        1.  [new SftpClient(name) ===> SFTP client object](#orga990762)
        2.  [connect(config) ===> SFTPstream](#orga24c78c)
        3.  [list(path, pattern) ==> Array[object]](#org62856bf)
        4.  [exists(path) ==> boolean](#orgf1725d7)
        5.  [stat(path) ==> object](#orgca82db5)
        6.  [get(path, dst, options) ==> String|Stream|Buffer](#orgf03d9dc)
        7.  [fastGet(remotePath, localPath, options) ===> string](#org60015f8)
        8.  [put(src, remotePath, options) ==> string](#org4c453c1)
        9.  [fastPut(localPath, remotePath, options) ==> string](#org43ba146)
        10. [append(input, remotePath, options) ==> string](#orgeb14ec0)
        11. [mkdir(path, recursive) ==> string](#orge36efb6)
        12. [rmdir(path, recursive) ==> string](#org4e9756a)
        13. [delete(path) ==> string](#org5558876)
        14. [rename(fromPath, toPath) ==> string](#org8d1ae7f)
        15. [chmod(path, mode) ==> string](#orgd6f506d)
        16. [realPath(path) ===> string](#orge278e16)
        17. [cwd() ==> string](#org03d8066)
        18. [end() ==> boolean](#org48ede29)
        19. [Add and Remove Listeners](#orgdb4973a)
8.  [FAQ](#orgd9bd9d8)
    1.  [Remote server drops connections with only an end event](#org5abb622)
    2.  [How can you pass writable stream as dst for get method?](#org3e21a0f)
    3.  [How can I upload files without having to specify a password?](#orgad53a21)
    4.  [How can I connect through a Socks Proxy](#org89b487a)
9.  [Change Log](#org74bc7f8)
    1.  [v4.2.1 (Prod Version)](#org135b786)
    2.  [v4.2.0](#orgec101e3)
    3.  [v4.1.0](#org5da0e74)
    4.  [v4.0.4](#org8266de9)
    5.  [v4.0.3](#orgef2b0c4)
    6.  [v4.0.2](#org9e36bec)
    7.  [v4.0.0](#orge515716)
    8.  [v2.5.2](#org453ef69)
    9.  [v2.5.1](#org929e671)
    10. [v2.5.0](#org09654d5)
    11. [v2.4.3](#orgc17457a)
    12. [v2.4.2](#org3708b37)
    13. [v2.4.1](#org5293271)
    14. [v2.4.0](#orgaea5aba)
    15. [v2.3.0](#org65eab1d)
    16. [v3.0.0 &#x2013; deprecate this version](#orgcb65936)
    17. [v2.1.1](#org5324914)
    18. [v2.0.1](#org7238a3b)
    19. [v1.1.0](#org026a96e)
    20. [v1.0.5:](#org0e31491)
10. [Logging Issues](#org4860a8f)
11. [Pull Requests](#orgffb1857)
12. [Contributors](#org891dcee)


<a id="org2ff92dd"></a>

# SSH2 SFTP Client

an SFTP client for node.js, a wrapper around [SSH2](https://github.com/mscdex/ssh2)  which provides a high level
convenience abstraction as well as a Promise based API.

Documentation on the methods and available options in the underlying modules can
be found on the [SSH2](https://github.com/mscdex/ssh2) and [SSH2-STREAMS](https://github.com/mscdex/ssh2-streams/blob/master/SFTPStream.md)  project pages.

Current stable release is **v4.1.0**.

Code has been tested against Node versions 8.16.1, 10.16.3 and 12.9.1

Node versions < 8.x are not supported.


<a id="org101da24"></a>

# Installation

    npm install ssh2-sftp-client


<a id="orge0baaf0"></a>

# Basic Usage

    let Client = require('ssh2-sftp-client');
    let sftp = new Client();
    
    sftp.connect({
      host: '127.0.0.1',
      port: '8080',
      username: 'username',
      password: '******'
    }).then(() => {
      return sftp.list('/pathname');
    }).then(data => {
      console.log(data, 'the data info');
    }).catch(err => {
      console.log(err, 'catch error');
    });


<a id="org1450b73"></a>

# Breaking Changes in Version 4.x

There has been minor changes to the API signatures

-   The `connect()` method no longer accepts a 'connectMethod' argument. It was
    not clear what this argument was for or what it did.

-   Additional options are now available in the configure object passed to the
    `connect()` method to control the connection retry functionality.

-   Node versions before 8.x are no longer supported.

-   Error message formats have changed. While they are now more consistent, if you
    have code which parses the messages, it will need to be updated.

-   The `auxList()` method is deprecated. An additional optional `pattern`
    argument has been added to the `list()` method to facilitate filtering of
    results returned by `list()`. Both 'glob' and regexp pattern styles are
    supported.

-   The properties returned by the `stat()` method have changed. The `permissions`
    property has been removed as it contained the same information as the `mode`
    property. New properties `isDirectory`, `isFile`, `isBlockDevice`,
    `isCharacterDevice`, `isSymbolicLink`, `isFIFO` and `isSocket` have been added.


<a id="org0da6002"></a>

# Enhancements in Version 4.2.x

-   Added ability to set a client name in `Client()` constructor. This can be
    useful when creating multiple clients as the client name will be displayed
    in error messages, providing a clue as to which client has failed.
-   Added a work-around for the SSH2 bug which results in only an `end` event
    being raised when the remote server drops the connection during the connect
    process. (see FAQ section below).
-   Added additional error checking to prevent attempts to call `connect()` on
    an already connected client. While a client can be used to make multiple
    connections, you must call `end()` before calling `connect()` again. Each
    client object can only represent a single connection to an SFTP
    server. However, some methods, such as `fastPut()` and `fastGet()` will use
    concurrency to speed up the transfer of data.
-   Added some more examples in the example directory of the repository


<a id="org436dffc"></a>

# Enhancements in Version 4.1.x

-   Some of the data upload/download methods would create an empty destination
    file when the source file did not exist. This has now been fixed.
-   Handling of relative path names was weak and inconsistent. This has now been
    made more consistent and reliable. Two new methods `realPath()` and `cwd()`
    have been added
-   More error checking and provision of error messages with more meaningful
    information. Expansion and enhancements of test cases.


<a id="org627f927"></a>

# Documentation

The connection options are the same as those offered by the underlying SSH2
module. For full details, please see [SSH2 client methods](https://github.com/mscdex/ssh2#user-content-client-methods)

All the methods will return a Promise, except for `on()` and
`removeListener()`, which are typically only used in special use cases. 


<a id="org2e64c16"></a>

## Methods


<a id="orga990762"></a>

### new SftpClient(name) ===> SFTP client object

Constructor to create a new `ssh2-sftp-client` object. An optional `name` string
can be provided, which will be used in error messages to help identify which
client has thrown the error. 

1.  Constructor Arguments

    -   **name:** string. An optional name string used in error messages

2.  Example Use

        'use strict';
        
        const Client = require('ssh2-sftp-client');
        
        const config = {
          host: 'example.com',
          user: 'donald',
          password: 'my-secret'
        };
        
        const sftp = new Client('example-client');
        
        sftp.connect(config)
          .then(() => {
            return sftp.cwd();
          })
          .then(p => {
            console.log(`Remote working directory is ${p}`);
            return sftp.end();
          })
          .catch(err => {
            console.log(`Error: ${err.message}`); // error message will include 'example-client'
          });


<a id="orga24c78c"></a>

### connect(config) ===> SFTPstream

Connect to an sftp server. Full documentation for connection options is
available [here](https://github.com/mscdex/ssh2#user-content-client-methods)

1.  Connection Options

    This module is based on the excellent [SSH2](https://github.com/mscdex/ssh2#client) module. That module is a general SSH2
    client and server library and provides much more functionality than just SFTP
    connectivity. Many of the connect options provided by that module are less
    relevant for SFTP connections. It is recommended you keep the config options to
    the minimum needed and stick to the options listed in the `commonOpts` below. 
    
    The `retries`, `retry_factor` and `retry_minTimeout` options are not part of the
    SSH2 module. These are part of the configuration for the [retry](https://www.npmjs.com/package/retry) package and what
    is used to enable retrying of sftp connection attempts. See the documentation
    for that package for an explanation of these values.  
    
        // common options
        
        let commonOpts {
          host: 'localhost', // string Hostname or IP of server.
          port: 22, // Port number of the server.
          forceIPv4: false, // boolean (optional) Only connect via IPv4 address
          forceIPv6: false, // boolean (optional) Only connect via IPv6 address
          username: 'donald', // string Username for authentication.
          password: 'borsch', // string Password for password-based user authentication
          agent: process.env.SSH_AGENT, // string - Path to ssh-agent's UNIX socket
          privateKey: fs.readFileSync('/path/to/key'), // Buffer or string that contains
          passphrase; 'a pass phrase', // string - For an encrypted private key
          readyTimeout: 20000, // integer How long (in ms) to wait for the SSH handshake
          strictVendor: true // boolean - Performs a strict server vendor check
          debug: myDebug // function - Set this to a function that receives a single
                        // string argument to get detailed (local) debug information. 
          retries: 2 // integer. Number of times to retry connecting
          retry_factor: 2 // integer. Time factor used to calculate time between retries
          retry_minTimeout: 2000 // integer. Minimum timeout between attempts
        };
        
        // rarely used options
        
        let advancedOpts {
          localAddress,
          localPort,
          hostHash,
          hostVerifier,
          agentForward,
          localHostname,
          localUsername,
          tryKeyboard,
          authHandler,
          keepaliveInterval,
          keepaliveCountMax,
          sock,
          algorithms,
          compress
        };

2.  Example Use

        sftp.connect({
          host: example.com,
          port: 22,
          username: 'donald',
          password: 'youarefired'
        });


<a id="org62856bf"></a>

### list(path, pattern) ==> Array[object]

Retrieves a directory listing. This method returns a Promise, which once
realised, returns an array of objects representing items in the remote
directory. 

-   **path:** {String} Remote directory path
-   **pattern:** (optional) {string|RegExp} A pattern used to filter the items included in the returned
    array. Pattern can be a simple *glob*-style string or a regular
    expression. Defaults to `/.*/`.

1.  Example Use

        const Client = require('ssh2-sftp-client');
        
        const config = {
          host: 'example.com',
          port: 22,
          username: 'red-don',
          password: 'my-secret'
        };
        
        let sftp = new Client;
        
        sftp.connect(config)
          .then(() => {
            return sftp.list('/path/to/remote/dir');
          })
          .then(data => {
            console.log(data);
          })
          .then(() => {
            sftp.end();
          })
          .catch(err => {
            console.error(err.message);
          });

2.  Return Objects

    The objects in the array returned by `list()` have the following properties;
    
        {
          type: // file type(-, d, l)
          name: // file name
          size: // file size
          modifyTime: // file timestamp of modified time
          accessTime: // file timestamp of access time
          rights: {
            user:
            group:
            other:
          },
          owner: // user ID
          group: // group ID
        }

3.  Pattern Filter

    The filter options can be a regular expression (most powerful option) or a
    simple *glob*-like string where \* will match any number of characters, e.g.
    
        foo* => foo, foobar, foobaz
        *bar => bar, foobar, tabbar
        *oo* => foo, foobar, look, book
    
    The *glob*-style matching is very simple. In most cases, you are best off using
    a real regular expression which will allow you to do more powerful matching and
    anchor matches to the beginning/end of the string etc.


<a id="orgf1725d7"></a>

### exists(path) ==> boolean

Tests to see if remote file or directory exists. Returns type of remote object
if it exists or false if it does not.

1.  Example Use

        const Client = require('ssh2-sftp-client');
        
        const config = {
          host: 'example.com',
          port: 22,
          username: 'red-don',
          password: 'my-secret'
        };
        
        let sftp = new Client;
        
        sftp.connect(config)
          .then(() => {
            return sftp.exists('/path/to/remote/dir');
          })
          .then(data => {
            console.log(data);          // will be false or d, -, l (dir, file or link)
          })
          .then(() => {
            sftp.end();
          })
          .catch(err => {
            console.error(err.message);
          });


<a id="orgca82db5"></a>

### stat(path) ==> object

Returns the attributes associated with the object pointed to by `path`. 

-   **path:** String. Remote path to directory or file on remote server

1.  Attributes

    The `stat()` method returns an object with the following properties;
    
        let stats = {
          mode: 33279, // integer representing type and permissions
          uid: 1000, // user ID
          gid: 985, // group ID
          size: 5, // file size
          accessTime: 1566868566000, // Last access time. milliseconds
          modifyTime: 1566868566000, // last modify time. milliseconds
          isDirectory: false, // true if object is a directory
          isFile: true, // true if object is a file
          isBlockDevice: false, // true if object is a block device
          isCharacterDevice: false, // true if object is a character device
          isSymbolicLink: false, // true if object is a symbolic link
          isFIFO: false, // true if object is a FIFO
          isSocket: false // true if object is a socket
        };

2.  Example Use

        let client = new Client();
        
        client.connect(config)
          .then(() => {
            return client.stat('/path/to/remote/file');
          })
          .then(data => {
            // do something with data
          })
          .then(() => {
            client.end();
          })
          .catch(err => {
            console.error(err.message);
          });


<a id="orgf03d9dc"></a>

### get(path, dst, options) ==> String|Stream|Buffer

Retrieve a file from a remote SFTP server. The `dst` argument defines the
destination and can be either a string, a writeable stream or undefined. If it
is a string, it is interpreted as the path to a location on the local file
system (path should include the file name). If it is a writeable stream, the
remote data is piped into that stream. If `dst` is undefined, the method will
put the data into a buffer and return that buffer when the Promise is
resolved. If `dst` is defined, it is returned when the Promise is resolved.

In general, if your going to pass in a string as the destination, you are
probably better off using the `fastGet()` method.

-   **path:** String. Path to the remote file to download
-   **dst:** String|Stream. Destination for the data. If a string, it
    should be a local file path.
-   **options:** Options for the `get()` command (see below).

1.  Options

    The options object can be used to pass options to the underlying readStream used
    to read the data from the remote server. 
    
        { flags: 'r',
          encoding: null,
          handle: null,
          mode: 0o666,
          autoClose: true
        }
    
    Most of the time, you won't want to use any options. Sometimes, it may be useful
    to set the encoding. For example, to 'utf-8'. However, it is important not to do
    this for binary files to avoid data corruption. 

2.  Example Use

        let client = new Client();
        
        let remotePath = '/remote/server/path/file.txt';
        let dst = fs.createWriteStream('/local/file/path/copy.txt');
        
        client.connect(config)
          .then(() => {
            return client.get(remotePath, dst);
          })
          .then(() => {
            client.end();
          })
          .catch(err => {
            console.error(err.message);
          });
    
    -   **Tip:** See examples file in the Git repository for more examples. You can pass
        any writeable stream in as the destination. For example, if you pass in
        `zlib.createGunzip()` writeable stream, you can both download and
        decompress a gzip file 'on the fly'.


<a id="org60015f8"></a>

### fastGet(remotePath, localPath, options) ===> string

Downloads a file at remotePath to localPath using parallel reads for faster
throughput. This is the simplest method if you just want to download a file.

-   **remotePath:** String. Path to the remote file to download
-   **localPath:** String. Path on local file system for the downloaded file. The
    local path should include the filename to use for saving the
    file.
-   **options:** Options for `fastGet()` (see below)

1.  Options

        {
          concurrency: 64, // integer. Number of concurrent reads to use
          chunkSize: 32768, // integer. Size of each read in bytes
          step: function(total_transferred, chunk, total) // callback called each time a
           // chunk is transferred
        }
    
    -   **Warning:** Some servers do not respond correctly to requests to alter chunk
        size. This can result in lost or corrupted data.

2.  Sample Use

        let client = new Client();
        let remotePath = '/server/path/file.txt';
        let localPath = '/local/path/file.txt';
        
        client.connect(config)
          .then(() => {
            client.fastGet(remotePath, localPath);
          })
          .then(() => {
            client.end();
          })
          .catch(err => {
            console.error(err.message);
          });


<a id="org4c453c1"></a>

### put(src, remotePath, options) ==> string

Upload data from local system to remote server. If the `src` argument is a
string, it is interpreted as a local file path to be used for the data to
transfer. If the `src` argument is a buffer, the contents of the buffer are
copied to the remote file and if it is a readable stream, the contents of that
stream are piped to the `remotePath` on the server. 

-   **src:** string | buffer | readable stream. Data source for data to copy to the
    remote server.
-   **remotePath:** string. Path to the remote file to be created on the server.
-   **options:** object. Options which can be passed to adjust the write stream used
    in sending the data to the remote server (see below).

1.  Options

    The following options are supported;
    
        {
          flags: 'w',  // w - write and a - append
          encoding: null, // use null for binary files
          mode: 0o666, // mode to use for created file (rwx)
          autoClose: true // automatically close the write stream when finished
        }
    
    The most common options to use are mode and encoding. The values shown above are
    the defaults. You do not have to set encoding to utf-8 for text files, null is
    fine for all file types. However, using utf-8 encoding for binary files will
    often result in data corruption. 

2.  Example Use

        let client = new Client();
        
        let data = fs.createReadStream('/path/to/local/file.txt');
        let remote = '/path/to/remote/file.txt';
        
        client.connect(config)
          .then(() => {
            return client.put(data, remote);
          })
          .then(() => {
            return client.end();
          })
          .catch(err => {
            console.error(err.message);
          });
    
    -   **Tip:** If the src argument is a path string, consider just using `fastPut()`.


<a id="org43ba146"></a>

### fastPut(localPath, remotePath, options) ==> string

Uploads the data in file at `localPath` to a new file on remote server at
`remotePath` using concurrency. The options object allows tweaking of the fast put process. 

-   **localPath:** string. Path to local file to upload
-   **remotePath:** string. Path to remote file to create
-   **options:** object. Options passed to createWriteStream (see below)

1.  Options

        {
          concurrency: 64, // integer. Number of concurrent reads
          chunkSize: 32768, // integer. Size of each read in bytes
          mode: 0o755, // mixed. Integer or string representing the file mode to set
          step: function(total_transferred, chunk, total) // function. Called every time
          // a part of a file was transferred
        }
    
    -   **Warning:** There have been reports that some SFTP servers will not honour
        requests for non-default chunk sizes. This can result in data loss
        or corruption.

2.  Example Use

        let localFile = '/path/to/file.txt';
        let remoteFile = '/path/to/remote/file.txt';
        let client = new Client();
        
        client.connect(config)
          .then(() => {
            client.fastPut(localFile, remoteFile);
          })
          .then(() => {
            client.end();
          })
          .catch(err => {
            console.error(err.message);
          });


<a id="orgeb14ec0"></a>

### append(input, remotePath, options) ==> string

Append the `input` data to an existing remote file. There is no integrity
checking performed apart from normal writeStream checks. This function simply
opens a writeStream on the remote file in append mode and writes the data passed
in to the file. 

-   **input:** buffer | readStream. Data to append to remote file
-   **remotePath:** string. Path to remote file
-   **options:** object. Options to pass to writeStream (see below)

1.  Options

    The following options are supported;
    
        {
          flags: 'a',  // w - write and a - append
          encoding: null, // use null for binary files
          mode: 0o666, // mode to use for created file (rwx)
          autoClose: true // automatically close the write stream when finished
        }
    
    The most common options to use are mode and encoding. The values shown above are
    the defaults. You do not have to set encoding to utf-8 for text files, null is
    fine for all file types. Generally, I would not attempt to append binary files. 

2.  Example Use

        let remotePath = '/path/to/remote/file.txt';
        let client = new Client();
        
        client.connect(config)
          .then(() => {
            return client.append(Buffer.from('Hello world'), remotePath);
          })
          .then(() => {
            return client.end();
          })
          .catch(err => {
            console.error(err.message);
          });


<a id="orge36efb6"></a>

### mkdir(path, recursive) ==> string

Create a new directory. If the recursive flag is set to true, the method will
create any directories in the path which do not already exist. Recursive flag
defaults to false.

-   **path:** string. Path to remote directory to create
-   **recursive:** boolean. If true, create any missing directories in the path as
    well

1.  Example Use

        let remoteDir = '/path/to/new/dir';
        let client = new Client();
        
        client.connect(config)
          .then(() => {
            return client.mkdir(remoteDir, true);
          })
          .then(() => {
            return client.end();
          })
          .catch(err => {
            console.error(err.message);
          });


<a id="org4e9756a"></a>

### rmdir(path, recursive) ==> string

Remove a directory. If removing a directory and recursive flag is set to
`true`, the specified directory and all sub-directories and files will be
deleted. If set to false and the directory has sub-directories or files, the
action will fail. 

-   **path:** string. Path to remote directory
-   **recursive:** boolean. If true, remove all files and directories in target
    directory. Defaults to false

1.  Example Use

        let remoteDir = '/path/to/remote/dir';
        let client = new Client();
        
        client.connect(config)
          .then(() => {
            return client.rmdir(remoteDir, true);
          })
          .then(() => {
            return client.end();
          })
          .catch(err => {
            console.error(err.message);
          });


<a id="org5558876"></a>

### delete(path) ==> string

Delete a file on the remote server.

-   **path:** string. Path to remote file to be deleted.

1.  Example Use

        let remoteFile = '/path/to/remote/file.txt';
        let client = new Client();
        
        client.connect(config)
          .then(() => {
            return client.delete(remoteFile);
          })
          .then(() => {
            return client.end();
          })
          .catch(err => {
            console.error(err.message);
          });


<a id="org8d1ae7f"></a>

### rename(fromPath, toPath) ==> string

Rename a file or directory from `fromPath` to `toPath`. You must have the
necessary permissions to modify the remote file. 

1.  Example Use

        let from = '/remote/path/to/old.txt';
        let to = '/remote/path/to/new.txt';
        let client = new Client();
        
        client.connect(config)
          .then(() => {
            return client.rename(from, to);
          })
          .then(() => {
            return client.end();
          })
          .catch(err => {
            console.error(err.message);
          });


<a id="orgd6f506d"></a>

### chmod(path, mode) ==> string

Change the mode (read, write or execute permissions) of a remote file or
directory.

-   **path:** string. Path to the remote file or directory
-   **mode:** octal. New mode to set for the remote file or directory

1.  Example Use

        let path = '/path/to/remote/file.txt';
        let ndwMode = 0o644;  // rw-r-r
        let client = new Client();
        
        client.connect(config)
          .then(() => {
            return client.chmod(path, newMode);
          })
          .then(() => {
            return client.end();
          })
          .catch(err => {
            console.error(err.message);
          });


<a id="orge278e16"></a>

### realPath(path) ===> string

Converts a relative path to an absolute path on the remote server. This method
is mainly used internally to resolve remote path names. 

-   **path:** A file path, either relative or absolute


<a id="org03d8066"></a>

### cwd() ==> string

Returns what the server believes is the current remote working directory. 


<a id="org48ede29"></a>

### end() ==> boolean

Ends the current client session, releasing the client socket and associated
resources. This function also removes all listeners associated with the client.

1.  Example Use

        let client = new Client();
        
        client.connect(config)
          .then(() => {
            // do some sftp stuff
          })
          .then(() => {
            return client.end();
          })
          .catch(err => {
            console.error(err.message);
          });


<a id="orgdb4973a"></a>

### Add and Remove Listeners

Although normally not required, you can add and remove custom listeners on the
ssh2 client object. This object supports a number of events, but only a few of
them have any meaning in the context of SFTP. These are 

-   **error:** An error occurred. Calls listener with an error argument.
-   **end:** The socket has been disconnected. No argument.
-   **close:** The socket was closed. Boolean argument which is true when the socket
    was closed due to errors.

1.  on(eventType, listener)

    Adds the specified listener to the specified event type. It the event type is
    `error`, the listener should accept 1 argument, which will be an Error object. If
    the event type is `close`, the listener should accept one argument of a boolean
    type, which will be true when the client connection was closed due to errors.

2.  removeListener(eventType, listener)

    Removes the specified listener from the event specified in eventType. Note that
    the `end()` method automatically removes all listeners from the client object. 


<a id="orgd9bd9d8"></a>

# FAQ


<a id="org5abb622"></a>

## Remote server drops connections with only an end event

Many SFTP servers have rate limiting protection which will drop connections once
a limit has been reached. In particular, openSSH has the setting `MaxStartups`,
which can be a tuple of the form `max:drop:full` where `max` is the maximum
allowed unauthenticated connections, `drop` is a percentage value which
specifies percentage of connections to be dropped once `max` connections has
been reached and `full` is the number of connections at which point all
subsequent connections will be dropped. e.g. `10:30:60` means allow up to 10
unauthenticated connections after which drop 30% of connection attempts until
reaching 60 unauthenticated connections, at which time, drop all attempts. 

Clients first make an unauthenticated connection to the SFTP server to begin
negotiation of protocol settings (cipher, authentication method etc). If you are
creating multiple connections in a script, it is easy to exceed the limit,
resulting in some connections being dropped. As SSH2 only raises an 'end' event
for these dropped connections, no error is detected. The `ssh2-sftp-client` now
listens for `end` events during the connection process and if one is detected,
will reject the connection promise. 

One way to avoid this type of issue is to add a delay between connection
attempts. It does not need to be a very long delay - just sufficient to permit
the previous connection to be authenticated. In fact, the default setting for
openSSH is `10:30:60`, so you really just need to have enough delay to ensure
that the 1st connection has completed authentication before the 11th connection
is attempted. 


<a id="org3e21a0f"></a>

## How can you pass writable stream as dst for get method?

If the dst argument passed to the get method is a writeable stream, the remote
file will be piped into that writeable. If the writeable you pass in is a
writeable stream created with `fs.createWriteStream()`, the data will be written
to the file specified in the constructor call to `createWriteStream()`. 

The writeable stream can be any type of write stream. For example, the below code
will convert all the characters in the remote file to upper case before it is
saved to the local file system. This could just as easily be something like a
gunzip stream from `zlib`, enabling you to decompress remote zipped files as you
bring them across before saving to local file system. 

    'use strict';
    
    // Example of using a writeable with get to retrieve a file.
    // This code will read the remote file, convert all characters to upper case
    // and then save it to a local file
    
    const Client = require('../src/index.js');
    const path = require('path');
    const fs = require('fs');
    const through = require('through2');
    
    const config = {
      host: 'arch-vbox',
      port: 22,
      username: 'tim',
      password: 'xxxx'
    };
    
    const sftp = new Client();
    const remoteDir = '/home/tim/testServer';
    
    function toupper() {
      return through(function(buf, enc, next) {
        next(null, buf.toString().toUpperCase());
      });
    }
    
    sftp
      .connect(config)
      .then(() => {
        return sftp.list(remoteDir);
      })
      .then(data => {
        // list of files in testServer
        console.dir(data);
        let remoteFile = path.join(remoteDir, 'test.txt');
        let upperWtr = toupper();
        let fileWtr = fs.createWriteStream(path.join(__dirname, 'loud-text.txt'));
        upperWtr.pipe(fileWtr);
        return sftp.get(remoteFile, upperWtr);
      })
      .then(() => {
        return sftp.end();
      })
      .catch(err => {
        console.error(err.message);
      });


<a id="orgad53a21"></a>

## How can I upload files without having to specify a password?

There are a couple of ways to do this. Essentially, you want to setup SSH keys
and use these for authentication to the remote server. 

One solution, provided by @KalleVuorjoki is to use the SSH agent
process. **Note**: SSH<sub>AUTH</sub><sub>SOCK</sub> is normally created by your OS when you load the
ssh-agent as part of the login session. 

    let sftp = new Client();
    sftp.connect({
      host: 'YOUR-HOST',
      port: 'YOUR-PORT',
      username: 'YOUR-USERNAME',
      agent: process.env.SSH_AUTH_SOCK
    }).then(() => {
      sftp.fastPut(....)
    }

Another alternative is to just pass in the SSH key directly as part of the
configuration. 

    let sftp = new Client();
    sftp.connect({
      host: 'YOUR-HOST',
      port: 'YOUR-PORT',
      username: 'YOUR-USERNAME',
      privateKey: fs.readFileSync('/path/to/ssh/key')
    }).then(() => {
      sftp.fastPut(.....)
    }


<a id="org89b487a"></a>

## How can I connect through a Socks Proxy

This solution was provided by @jmorino. 

    import { SocksClient } from 'socks';
    import SFTPClient from 'ssh2-sftp-client';
    
    const host = 'my-sftp-server.net';
    const port = 22; // default SSH/SFTP port on remote server
    
    // connect to SOCKS 5 proxy
    const { socket } = await SocksClient.createConnection({
      proxy: {
        host: 'my.proxy', // proxy hostname
        port: 1080, // proxy port
        type: 5, // for SOCKS v5
      },
      command: 'connect',
      destination: { host, port } // the remote SFTP server
    });
    
    const client = new SFTPClient();
    client.connect({
      host,
      sock: socket, // pass the socket to proxy here (see ssh2 doc)
      username: '.....',
      privateKey: '.....'
    })
    
    
    // client is connected


<a id="org74bc7f8"></a>

# Change Log


<a id="org135b786"></a>

## v4.2.1 (Prod Version)

-   Remove default close listener. changes in ssh2 API removed the utility of a
    default close listener
-   Fix path handling. Under mixed environments (where client platform and
    server platform were different i.e. one windows the other unix), path
    handling was broken due tot he use of path.join().
-   Ensure error messages include path details. Instead of errors such as "No
    such file" now report "No such file /path/to/missing/file" to help with
    debugging


<a id="orgec101e3"></a>

## v4.2.0

-   Work-around for SSH2 `end` event bug
-   Added ability to set client name in constructor method
-   Added additional error checking to prevent `connect()` being called on
    already connected client
-   Added additional examples in `example` directory


<a id="org5da0e74"></a>

## v4.1.0

-   move `end()` call to resolve into close hook
-   Prevent `put()` and `get()` from creating empty files in destination when
    unable to read source
-   Expand tests for operations when lacking required permissions
-   Add additional data checks for `append()`
    -   Verify file exists
    -   Verify file is writeable
    -   Verify file is a regular file
-   Fix handling of relative paths
-   Add `realPath()` method
-   Add `cwd()` method


<a id="org8266de9"></a>

## v4.0.4

-   Minor documentation fix
-   Fix return value from `get()`


<a id="orgef2b0c4"></a>

## v4.0.3

-   Fix bug in mkdir() relating to handling of relative paths
-   Modify exists() to always return 'd' if path is '.'


<a id="org9e36bec"></a>

## v4.0.2

-   Fix some minor packaging issues


<a id="orge515716"></a>

## v4.0.0

-   Remove support for node < 8.x
-   Fix connection retry feature
-   sftp connection object set to null when 'end' signal is raised
-   Removed 'connectMethod' argument from connect method.
-   Refined adding/removing of listeners in connect() and end() methods to enable
    errors to be adequately caught and reported.
-   Deprecate auxList() and add pattern/regexp filter option to list()
-   Refactored handling of event signals to provide better feedback to clients
-   Removed pointless 'permissions' property from objects returned by `stat()`
    (same as mode property). Added additional properties describing the type of
    object.
-   Added the `removeListener()` method to compliment the existing `on()` method.


<a id="org453ef69"></a>

## v2.5.2

-   Repository transferred to theophilusx
-   Fix error in package.json pointing to wrong repository


<a id="org929e671"></a>

## v2.5.1

-   Apply 4 pull requests to address minor issues prior to transfer


<a id="org09654d5"></a>

## v2.5.0

-   ???


<a id="orgc17457a"></a>

## v2.4.3

-   merge #108, #110
    -   fix connect promise if connection ends


<a id="org3708b37"></a>

## v2.4.2

-   merge #105
    -   fix windows path


<a id="org5293271"></a>

## v2.4.1

-   merge pr #99, #100
    -   bug fix


<a id="orgaea5aba"></a>

## v2.4.0

-   Requires node.js v7.5.0 or above.
-   merge pr #97, thanks for @theophilusx
    -   Remove emitter.maxListener warnings
    -   Upgraded ssh2 dependency from 0.5.5 to 0.6.1
    -   Enhanced error messages to provide more context and to be more consistent
    -   re-factored test
    -   Added new 'exists' method and re-factored mkdir/rmdir


<a id="org65eab1d"></a>

## v2.3.0

-   add: \`stat\` method
-   add \`fastGet\` and \`fastPut\` method.
-   fix: \`mkdir\` file exists decision logic


<a id="orgcb65936"></a>

## v3.0.0 &#x2013; deprecate this version

-   change: \`sftp.get\` will return chunk not stream anymore
-   fix: get readable not emitting data events in node 10.0.0


<a id="org5324914"></a>

## v2.1.1

-   add: event listener. [doc](<https://github.com/jyu213/ssh2-sftp-client#Event>)
-   add: \`get\` or \`put\` method add extra options [pr#52](<https://github.com/jyu213/ssh2-sftp-client/pull/52>)


<a id="org7238a3b"></a>

## v2.0.1

-   add: \`chmod\` method [pr#33](<https://github.com/jyu213/ssh2-sftp-client/pull/33>)
-   update: upgrade ssh2 to V0.5.0 [pr#30](<https://github.com/jyu213/ssh2-sftp-client/pull/30>)
-   fix: get method stream error reject unwork [#22](<https://github.com/jyu213/ssh2-sftp-client/issues/22>)
-   fix: return Error object on promise rejection [pr#20](<https://github.com/jyu213/ssh2-sftp-client/pull/20>)


<a id="org026a96e"></a>

## v1.1.0

-   fix: add encoding control support for binary stream


<a id="org0e31491"></a>

## v1.0.5:

-   fix: multi image upload
-   change: remove \`this.client.sftp\` to \`connect\` function


<a id="org4860a8f"></a>

# Logging Issues

Please log an issue for all bugs, questions, feature and enhancement
requests. Please ensure you include the module version, node version and
platform. 


<a id="orgffb1857"></a>

# Pull Requests

Pull requests are always welcomed. However, please ensure your changes pass all
tests and if your adding a new feature, that tests for that feature are
included. Likewise, for new features or enhancements, please include any
relevant documentation updates. 

This module will adopt a standard semantic versioning policy. Please indicate in
your pull request what level of change it represents i.e.

-   **Major:** Change to API or major change in functionality which will require an
    increase in major version number.
-   **Minor:** Minor change, enhancement or new feature which does not change
    existing API and will not break existing client code.
-   **Bug Fix:** No change to functionality or features. Simple fix of an existing
    bug.


<a id="org891dcee"></a>

# Contributors

This module was initially written by jyu213. On August 23rd, 2019, theophilusx
took over responsibility for maintaining this module. A number of other people
have contributed to this module, but until now, this was not tracked. My
intention is to credit anyone who contributes going forward. 

-   **jyu213:** Original author
-   **theophilusx:** Current maintainer
-   **henrytk:** Documentation fix

