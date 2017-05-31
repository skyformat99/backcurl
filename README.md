# backcurl
C++, pure libcurl based with customized set_easy_opt for different kind of requests for Mobile, NON BLOCK UI SYNC http request.



```
cd build
cmake ..
cmake --build .
ctest -VV
sudo make install
```



```
int main() {
	bcl::init(); // init when using

    bcl::execute<std::string>([&](bcl::Request &req) {
    	bcl::setOpts(req, CURLOPT_URL , "http://www.google.com",
                 CURLOPT_FOLLOWLOCATION, 1L,
                 CURLOPT_WRITEFUNCTION, &bcl::writeMemoryCallback,
                 CURLOPT_WRITEDATA, req.dataPtr,
                 CURLOPT_USERAGENT, "libcurl-agent/1.0",
                 CURLOPT_RANGE, "0-200000"
                );
	}, [&](bcl::Response & resp) {
        std::string ret =  std::string(resp.getBody<std::string>()->c_str());
        printf("Sync === %s\n", ret.c_str());
    });


    bcl::cleanUp(); // clean up when no more using
}
```


### for future non blocking sync 

```	
bcl::FutureResponse frp;

    frp = bcl::execFuture<std::string>(simpleGetOption);
    bool uiRunning = true;
    while (uiRunning)  {
        if (bcl::hasRequestedAndReady(frp)) {
            bcl::Response r = frp.get();
            printf("The data content is = %s\n", r.getBody<std::string>()->c_str());
            printf("Got Http Status code = %ld\n", r.code);
            printf("Got Content Type = %s\n", r.contentType.c_str());
            printf("Total Time Consume = %f\n", r.totalTime);
            printf("has Error = %s\n", !r.error.empty() ? "Yes" : "No");

            // Exit App
            uiRunning = false;
        }
        printf("\r Future Sync ==%s ----%d", "Drawing Graphiccccc with count elapsed ", countUI++);

    }
    
    if (!bcl::isProcessing(frp)) printf("no data process now, no more coming data\n\n" );
```    



### for run on ui and call back to Main thread
```
//please see the example/main.cpp doRunOnUI method
```


## for different kind of data type request by default is std::string based
```
struct myType {
	std::string s;
	float f;
};

size_t myMemoryCallBack(void *contents, size_t size, size_t nmemb, void *contentWrapper) {
    size_t realsize = size * nmemb;
    myType *memBlock = (myType *)contentWrapper;
    
    // do your jobs

    return realsize;
}


int main() {

	bcl::init(); // init when using

    bcl::execute<myType>([&](bcl::Request &req) {
    	bcl::setOpts(req, CURLOPT_URL , "http://www.google.com",
                 CURLOPT_FOLLOWLOCATION, 1L,
                 CURLOPT_WRITEFUNCTION, &myMemoryCallBack,
                 CURLOPT_WRITEDATA, req.dataPtr,
                 CURLOPT_USERAGENT, "libcurl-agent/1.0",
                 CURLOPT_RANGE, "0-200000"
                );
	}, [&](bcl::Response & resp) {
        myType* ret =  resp.getBody<myType>();
    });


    bcl::cleanUp(); // clean up when no more using
}
```


## Set Opts in 1 line to customized request
```
bcl::setOpts(req, CURLOPT_URL , "http://www.google.com",
                 CURLOPT_FOLLOWLOCATION, 1L,
                 CURLOPT_WRITEFUNCTION, &myMemoryCallBack,
                 CURLOPT_WRITEDATA, req.dataPtr,
                 CURLOPT_USERAGENT, "libcurl-agent/1.0",
                 CURLOPT_RANGE, "0-200000"
                );
```                
### for e.g, proxy request, proxy authenthication Get, Post, Head, Put, Basic Auth, please refer to libcurl for more details : https://curl.haxx.se/libcurl/c/curl_easy_setopt.html




## Set your http header in your request scope if needed, it is depended on libcurl CURLOPT_HTTPHEADER.
```
bcl::execute<myType>([&](bcl::Request &req) {
        req.headers->emplace_back("Authorization", res::mySetting[MY_BASIC_AUTH].asString());

        bcl::setOpts(req, CURLOPT_URL , "http://www.google.com",
                 CURLOPT_FOLLOWLOCATION, 1L,
                 CURLOPT_WRITEFUNCTION, &myMemoryCallBack,
                 CURLOPT_WRITEDATA, req.dataPtr,
                 CURLOPT_USERAGENT, "libcurl-agent/1.0",
                 CURLOPT_RANGE, "0-200000"
                );
    }, [&](bcl::Response & resp) {
        myType* ret =  resp.getBody<myType>();
    });
```


## Download image
### this needed to be done by wrapped file in the struct or class so it will be auto free the object. However, fd stream need to be closed at response scope
```
struct MyFileAgent {
    FILE *fd;
    struct stat fileInfo;
};

size_t write_data(void *ptr, size_t size, size_t nmemb, void *userData) {
    MyFileAgent* fAgent = (MyFileAgent*)userData;
    size_t written = fwrite((FILE*)ptr, size, nmemb, fAgent->fd);
    return written;
}
const char *url = "http://wallpapercave.com/wp/LmOgKXz.jpg";
    char outfilename[] = "husky_dog_wallpaper.jpeg";
    bcl::execute<MyFileAgent>([&](bcl::Request & req) -> void {
        MyFileAgent* fAgent = (MyFileAgent*)req.dataPtr;
        fAgent->fd = fopen(outfilename, "ab+");
        bcl::setOpts(req, CURLOPT_URL,url,
        CURLOPT_WRITEFUNCTION, write_data,
        CURLOPT_WRITEDATA, req.dataPtr,
        CURLOPT_FOLLOWLOCATION, 1L
                    );
    }, [&](bcl::Response & resp) {
        printf("Response content type = %s\n", resp.contentType.c_str());
        fclose(resp.getBody<MyFileAgent>()->fd);
    });

```



