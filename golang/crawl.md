## golang实现爬虫

```
package main

import (
 "fmt" 
)

type Fetcher interface {
    Fetcher(url string)(body string, urls []string, err error)
}

func Crawl(url string, depth int, fetcher Fetcher) {
    if depth <= 0 {
        return
    }

    body, urls, err := fetcher.Fetcher(url)
    if err != nil {
        fmt.Println(err)
        return
    }

    fmt.Printf("found: %s %q\n", url, body)

    for _, url := range urls {
        Crawl(url, depth - 1, fetcher)
    }

    return 
} 

func main(){
    Crawl("https://golang.org/", 4, fetcher)
}


type fakeFetcher map[string] *fakeResult

type fakeResult struct {
    body string
    urls []string
}

func (f fakeFetcher) Fetcher(url string)(body string, urls []string, err error) {
    if res, ok := f[url]; ok {
        return res.body, res.urls, nil
    }
    return "", nil, fmt.Errorf("not found: %s", url)
}

var fetcher = fakeFetcher{
    "https://golang.org/" : &fakeResult{
        "The Go Programming Language",
        []string{
            "https://golang.org/pkg/",
            "https://golang.org/cmd/",
        },
    },
    "https://golang.org/pkg/": &fakeResult{
        "Packages",
        []string{
            "https://golang.org/",
            "https://golang.org/cmd/",
            "https://golang.org/pkg/fmt/",
            "https://golang.org/pkg/os/",
        },
    },
    "https://golang.org/pkg/fmt/": &fakeResult{
        "Package fmt",
        []string{
            "https://golang.org/",
            "https://golang.org/pkg/",
        },
    },
    "https://golang.org/pkg/os/": &fakeResult{
        "Package os",
        []string{
            "https://golang.org/",
            "https://golang.org/pkg/",
        },
    },
}


```
