# Build and deploy
       
## Linux:

### Overview

The process consists of following steps:

1. Download libv8 monolith archive
2. Run the build
3. Deploy to GitHub Packages

### Download libv8 monolith archive

You can see which version is compatible with current source code by examining `v8/Dockerfile`.
Look for part that looks like this:

```docker
# Fetch V8 code
RUN fetch v8
WORKDIR /v8build/v8
RUN git checkout 9.3.345.11
```

The libv8 version we are interested in is `9.3.345.11`.

```bash
export LIB_V8_VERSION=9.3.345.11
curl -O https://download.eclipsesource.com/j2v8/v8/libv8_${LIB_V8_VERSION}_monolith.zip
rm -rf v8.out
mkdir -p v8.out
unzip libv8_${LIB_V8_VERSION}_monolith.zip -d v8.out
```

### Run the build

```bash
python2 build.py -t linux -a x64 --docker j2v8cmake j2v8jni j2v8cpp j2v8optimize j2v8java j2v8test
```

### Deploy to GitHub Packages

Since docker runs as root, first make sure that files have correct permissions:

```bash
sudo chown -R $(whoami) .
```

Make sure that you have correct credentials in `~/.m2/settings.xml`

```xml
<settings>
    <servers>
        <server>
            <id>github</id>
            <username>your-github-username</username>
            <password>personal-access-token-with-packages:write-permission</password>
        </server>
    </servers>
</settings>

```

Now we can deploy. 

```bash
mvn deploy
```
                   
## macOS

### Build on native macOS      

#### Run the build

You must build `V8` the same way as described in `provision_v8` step in `vagrant/macos/Vagrantfile`.
Next copy `include` directory and `out.gn/x64.release/obj/libv8_monolith.a` file to `j2v8/v8.out/`.

```bash
python2 build.py -t macos -a x64 j2v8cmake j2v8jni j2v8cpp j2v8optimize j2v8java j2v8test
```

### Build with Vagrant

#### Run the build

```bash
python2 build.py -t macos -a x64 -vgr --sys-image osx-catalina -nos j2v8cmake j2v8jni j2v8cpp j2v8optimize j2v8java j2v8test
```

The tricky part is to get the `osx-catalina` Vagrant image. It is not available anywhere, and you have to build it yourself.
It can be done with VirtualBox, but is very, very tricky.
Read [osx-catalina-box.md](osx-catalina-box.md).
                
#### Deploy to GitHub Packages

```bash
cd vagrant/macos
vagrant plugin install vagrant-scp
vagrant scp ~/.m2/settings.xml default:/Users/vagrant/.m2/
vagrant ssh -c "cd j2v8 && mvn deploy"
```
