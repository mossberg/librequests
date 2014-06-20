# librequests

Just because you're writing in C, doesn't mean everything has to be painful.

librequests is a wrapper library over [libcurl](http://curl.haxx.se/libcurl/)
which attempts to simplify submitting simple HTTP requests. It is influenced,
to a certain degree, by [python-requests](http://python-requests.org).

## dependencies

Make sure you have libcurl installed. On OS X, use [homebrew](http://brew.sh)
and run `brew install curl`.
On [Ubuntu](http://askubuntu.com/questions/78183/installing-curl-h-library)
`sudo apt-get install libcurl4-openssl-dev` will work.

## building

Simply run

```bash
$ make
```

This will create a `build/` directory which contains, `librequests.a`, a
static C library that you can compile your own code against.

## example

```c
#include <stdio.h>
#include "requests.h"

int main(int argc, const char *argv[])
{
    req_t req;                        /* declare struct used to store data */
    CURL *curl = requests_init(&req); /* setup */

    requests_get(curl, &req, "http://example.com"); /* submit GET request */
    printf("Request URL: %s\n", req.url);
    printf("Response Code: %lu\n", req.code);
    printf("Response Body:\n%s", req.text);

    requests_close(&req); /* clean up */
    return 0;
}
```

Save the above as `get.c`, and make sure `requests.h` (located in `include/`)
and `librequests.a` are in the same directory. Also make sure you have gcc or
equivalent and libcurl installed. Then compile it using:

```bash
$ gcc -o get get.c -L. -lrequests -lcurl
```

And you should see:

```bash
$ ./get
Request URL: http://example.com
Response Code: 200
Response Body:
<!doctype html>
<html>
<head>
    <title>Example Domain</title>
    ...
```

For more examples, look in the "examples" directory.

## documentation

Looking at the examples is probably the easiest way to learn to
use this. That said, there's a few core things that are good to know.

First off, all the good stuff happens with the `req_t` datatype which is just
a special struct that holds all the stuff from the request. Source below.

```
typedef struct {
    long code;           // Response Code
    char *url;           // Request URL
    char *text;          // Response Body
    size_t size;         // Length of Body
    char **headers;      // Character Array of Headers
    size_t headers_size; // Length of above array
    int ok;              // Bool value. Response codes < 400 are "ok"
} req_t;
```

At the beginning of every program that uses this library should be two lines.

```
req_t req;
CURL *curl = requests_init(&req);
```

The first line declares the `req_t` struct. The second line gets everything
set up and returns the curl handle that libcurl needs, which needs to be
passed into future functions.

At this point you're all set to actually make requests using the core
functions:

```
void requests_get(CURL *curl, req_t *req, char *url);
void requests_post(CURL *curl, req_t *req, char *url, char *data);
void requests_put(CURL *curl, req_t *req, char *url, char *data);
```

The data parameter of POST and PUT needs to already be url-encoded, but
the `requests_url_encode()` function can help you with that.

```
char *data[] = {
    "apple",  "red", // array of key-value pairs
    "banana", "yellow"
};
int data_size = sizeof(data)/sizeof(char*); // recommended way to get size
                                            // of array
...
char *body = requests_url_encode(curl, data, data_size);
requests_post(curl, &req, "http://www.posttestserver.com/post.php", body);
```

Lastly, make sure to call the cleanup functions once you're done. If you used
the url encode function, you'll need to separately `curl_free()` the returned
string, but otherwise, a simple call to `requests_close()` will do.

```
requests_close(&req);
```

## contribution

Feel free to fork this repo and tackle some of the
[issues](http://github.com/markmossberg/librequests/issues?page=1&state=open).

### tests

librequests uses the [greatest](https://github.com/silentbicycle/greatest) C
unit testing library, written by Scott Vokes. To compile and run tests, just
run

```bash
$ make test
$ ./test/test
```

To compile with debug statements, run

```bash
$ make test-debug
```

### debugging

`test.c` includes one of
[Zed's Awesome Debug Macros](http://c.learncodethehardway.org/book/ex20.html)
for debug statements that can easily be compiled in
or out. In your code, simply put `DEBUG("your message");` where the accepted
parameters are just like the ones for `printf()`.


[![Analytics](https://ga-beacon.appspot.com/UA-36552439-3/librequests/readme)](https://github.com/igrigorik/ga-beacon)
