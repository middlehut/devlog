### Building Btcd for Windows 10

The latest `btcd` release [build](https://github.com/btcsuite/btcd/releases) is from 2015. There's nice description how to build it for Linux/Mac, but for Windows it's suggested to download the latest binaries, which are 608 commits behind master as of December 2017. Fortunately, Golang can build binaries equally well on Windows, so we may follow the steps of the Linux build process. The only issue I faced was a bug in the glide dependency manager and that's how this little blog post is born.

*We assume that Golang is installed and GOPATH and GOROOT are configured.*

We can attempt to build and install it for Windows with the steps below:

```
c:\> go get -u github.com/Masterminds/glide
c:\> mkdir btcsuite
c:\> cd btcsuite
c:\btcsuite> git clone https://github.com/btcsuite/btcd
c:\btcsuite> cd btcd
c:\btcsuite\btcd> glide install
c:\btcsuite\btcd> go install . ./cmd/...
```
The issue happens on the line calling `glide install` and the error is something like: `Unable to export dependencies to vendor directory`

To fix it, we must change one line of code in `github.com/Masterminds/glide/path/winbug.go` and rebuild Glide.

In the function [**CustomRename**](https://github.com/Masterminds/glide/commit/cc37dc711a3191c2b91b01b9593c685660eeb9af), we have to replace   
```cmd := exec.Command("cmd.exe", "/c", "move", o, n)```   
with   
```cmd := exec.Command("cmd.exe", "/c", "xcopy /s/y", o, n+"\\")```   

Then change to the glide directory and rebuild it.
```
c:\> cd goprojects\src\github.com\Masterminds\glide
c:\goprojects\src\github.com\Masterminds\glide> go build
c:\goprojects\src\github.com\Masterminds\glide> go install
```

And we're ready to run `glide install` again in the btcd folder and everything should be OK.
