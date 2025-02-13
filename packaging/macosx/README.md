### How to make disk image with darktable application bundle (64 bit Intel only):

1. Install MacPorts ([instructions and prerequisites can be found on official website](https://www.macports.org/install.php), use Xcode version less than 12.2), please use default installation path (/opt/local).
2. Add some base configuration for macports
    ```bash
    cat <<EOF | sudo tee -a /opt/local/etc/macports/macports.conf
      macosx_deployment_target 10.7
      cxx_stdlib libc++
    EOF
    ```

    ```bash
    cat <<EOF | sudo tee -a /opt/local/etc/macports/variants.conf
      +no_gnome +no_x11 +quartz -x11 -gnome
    EOF
    ```
3. Install rsync port
    ```bash
    sudo port install -q rsync
    ```
4. Add mac ports patches for gnutls, python (https://trac.macports.org/ticket/51736), GraphicsMagick, ImageMagick, libusb, pugixml, librsvg, and gmic (all fixes except for gmic are required due to set deployment target, gmic patch removes extra dependencies):
    ```bash
     mkdir -p ~/ports/devel/gnutls/files ~/ports/lang/python{37,38,39}/files ~/ports/graphics/GraphicsMagick/files ~/ports/graphics/ImageMagick/files ~/ports/textproc ~/ports/science

     cp -R "$(port dir gnutls)" ~/ports/devel
     curl -Lo ~/ports/devel/gnutls/files/patch.diff https://raw.github.com/darktable-org/darktable/master/packaging/macosx/gnutls-disable-connectx.diff

     cp -R "$(port dir python37)" ~/ports/lang
     curl -Lo ~/ports/lang/python37/files/patch.diff https://raw.github.com/darktable-org/darktable/master/packaging/macosx/python36-stack_size.diff

     cp -R "$(port dir python38)" ~/ports/lang
     curl -Lo ~/ports/lang/python38/files/patch.diff https://raw.github.com/darktable-org/darktable/master/packaging/macosx/python38-stack_size.diff

     cp -R "$(port dir python39)" ~/ports/lang
     curl -Lo ~/ports/lang/python39/files/patch.diff https://raw.github.com/darktable-org/darktable/master/packaging/macosx/python38-stack_size.diff

     cp -R "$(port dir GraphicsMagick)" ~/ports/graphics
     curl -Lo ~/ports/graphics/GraphicsMagick/files/patch.diff https://raw.github.com/darktable-org/darktable/master/packaging/macosx/gm-deployment_target.diff

     cp -R "$(port dir ImageMagick)" ~/ports/graphics
     curl -Lo ~/ports/graphics/ImageMagick/files/patch.diff https://raw.github.com/darktable-org/darktable/master/packaging/macosx/im-stdlib.diff

     cp -R "$(port dir libusb)" ~/ports/devel
     curl -Lo ~/ports/devel/libusb/files/patch.diff https://raw.github.com/darktable-org/darktable/master/packaging/macosx/libusb-no-clock_gettime.diff

     cp -R "$(port dir pugixml)" ~/ports/textproc
     curl -L https://raw.github.com/darktable-org/darktable/master/packaging/macosx/pugixml-stdlib.patch | patch -d ~/ports -p0

     cp -R "$(port dir librsvg)" ~/ports/graphics
     curl -L https://raw.github.com/darktable-org/darktable/master/packaging/macosx/librsvg-nocargo.patch | patch -d ~/ports -p0

     cp -R "$(port dir gmic)" ~/ports/science
     curl -L https://raw.github.com/darktable-org/darktable/master/packaging/macosx/gmic-minimal.patch | patch -d ~/ports -p0
     ```
5. Append patchfile portfiles you just copied (except for pugixml, librsvg and gmic):
    ```bash
    echo 'patchfiles-append patch.diff' >> ~/ports/devel/gnutls/Portfile
    echo 'patchfiles-append patch.diff' >> ~/ports/devel/libusb/Portfile
    echo 'patchfiles-append patch.diff' >> ~/ports/graphics/GraphicsMagick/Portfile
    echo 'patchfiles-append patch.diff' >> ~/ports/graphics/ImageMagick/Portfile
    echo 'patchfiles-append patch.diff' >> ~/ports/lang/python37/Portfile
    echo 'patchfiles-append patch.diff' >> ~/ports/lang/python38/Portfile
    echo 'patchfiles-append patch.diff' >> ~/ports/lang/python39/Portfile
    ```

6. What does this do?
    ```bash
    portindex ~/ports
    ```

7. Add "file:///Users/<username>/ports" (change <username> to your actual login) to /opt/local/etc/macports/sources.conf before [default] line.
8. Install required dependencies (built from source):
    ```bash
    sudo port -q install -s git exiv2 libgphoto2 gtk-osx-application-gtk3 lensfun librsvg libsoup openexr json-glib flickcurl GraphicsMagick openjpeg lua webp libsecret pugixml osm-gps-map adwaita-icon-theme tango-icon-theme intltool iso-codes libomp gmic
    ```
9. Clone darktable git repository (in this example into ~/src):
    ```bash
    mkdir ~/src
    cd ~/src
    git clone --recurse-submodules https://github.com/darktable-org/darktable.git
    ```
10. Finally build and install darktable:
    ```bash
    cd darktable
    mkdir build
    cd build
    cmake .. -DCMAKE_OSX_DEPLOYMENT_TARGET=10.7 -DCMAKE_CXX_FLAGS=-stdlib=libc++ -DCMAKE_OBJCXX_FLAGS=-stdlib=libc++ -DOpenMP_C_INCLUDE_DIR=/opt/local/include/libomp -DOpenMP_CXX_INCLUDE_DIR=/opt/local/include/libomp -DCMAKE_LIBRARY_PATH=/opt/local/lib/libomp -DBINARY_PACKAGE_BUILD=ON -DRAWSPEED_ENABLE_LTO=ON -DBUILD_CURVE_TOOLS=ON -DBUILD_NOISE_TOOLS=ON
    make
    sudo make install
    ```
11. After this darktable will be installed in /usr/local directory and can be started by typing the following command in terminal:
    ```bash
     GSETTINGS_SCHEMA_DIR=/opt/local/share/glib-2.0/schemas/ XDG_DATA_DIRS=/opt/local/share darktable
     ```
12. Download, patch and install gtk-mac-bundler (assuming darktable was cloned into ~/src directory):
    ```bash
    cd ~/src
    curl -O https://ftp.gnome.org/pub/gnome/sources/gtk-mac-bundler/0.7/gtk-mac-bundler-0.7.4.tar.xz
    tar -xf gtk-mac-bundler-0.7.4.tar.xz
    cd gtk-mac-bundler-0.7.4
    patch -p1 < ../darktable/packaging/macosx/gtk-mac-bundler-0.7.4.patch
    make install
    ```
13. Now preparation is done, run image creating script, it should create darktable-<VERSION>.dmg in current (packaging/macosx) directory:
    ```bash
    cd ~/src/darktable/packaging/macosx
    ./make-app-bundle
    ```
14. If you have an Apple Developer ID Application certificate and want to produce signed app then replace the previous command with this one:
    ```bash
    CODECERT="Developer ID Application" APPLEID="developer@apple.id" APPLEPW="@keychain:DevIDPassword" ./make-app-bundle
    ```

    You may have to provide more specific name for a certificate if you have several matching in your keychain.

15. Assuming that darktable is the only program installed into /usr/local you should remove the install directory before doing next build:
    ```bash
    sudo rm -Rf /usr/local
    ```
