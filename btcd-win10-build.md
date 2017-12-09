## Building Btcd for Windows 10

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

### Note: If we want to store the blockchain on another drive than C:

If we want to change the default folder where `btcd` will download and store the blockchain, we can create a symbolic link, similar to `ln -s` in Linux. I wanted to change it, because I have a small 128GB SSD drive for booting Windows (C:) and 1TB spinning drive (D:) where I would like to store the data.

By default `btcd` stores the blockchain in `c:\Users\your-user\AppData\Local\Btcd\data`. If we want to move that directory to `D:` without disturbing the `btcd` expectations, we can create a symbolic link with the following commands:

First create a folder on drive (D:) where we'll store the blockchain, for example:
```
d:\> mkdir btcd-data
```
Next, we must rename(move) the current `data` folder created by `btcd`:
```
c:\Users\your-user\AppData\Local\Btcd> move data data-original
```
And create the symbolic link:
```
c:\Users\your-user\AppData\Local\Btcd> mklink /d "data" "d:\btcd-data"
```

Now, from `btcd` perspective it will have its `data` directory where it expects it to be, but the blockchain data will actually be stored on drive D: in the `d:\btcd-data` folder.
