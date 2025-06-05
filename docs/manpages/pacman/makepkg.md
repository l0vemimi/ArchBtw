# Creating a Package Repository

**Skip to** [4. Tarball](#4-tarball) **for a quick recap** on repo-add.

## Configuration: First Run

First and most obvious step is creating the GitHub page... This goes without saying so I'll keep it brief.

1. Create repository.
2. Enable GitHub pages & *deploy from branch*.

### 1. Keys

#### 1.1. Gpg Key

Create a gpg key:

        gpg --full-generate-key

And then to check to make sure it's been created correctly:

        gpg --list-key

*Take note of the email used and key id: Should contain 40 characters.*

Send your gpg key to a server to make sure it's working correctly.

        gpg --send-keys [GPG-KEY]

If successful you should then be able to recieve the key using:

        gpg --recv-keys [GPG-KEY]

#### 1.2. Pacman Key

Configure pacman to use the gpg key:

        sudo pacman-key --recv-keys [GPG-KEY]

And then locally sign the key:

        sudo pacman-key --lsign-key [GPG-KEY]

To build packages signed with this key it needs to be configured in makepkg in */etc/makepkg.conf*

        #-- Packager: name/email of the person or organization building packages
        PACKAGER="[NAME] <EMAIL>"
        #-- Specify a key to use for package signing
        GPGKEY="[GPG-KEY]"

### 2. Pacman Config

Add your repository to pacman in */etc/pacman.conf*. Add *TrustAll* and the repository server.

                [txted-repo]
                SigLevel = Optional TrustAll
                Server = https://[USERNAME].github.io/[REPOSITORY]/repo/x86_64

### 3. Repository

Create the repository directories:

        mkdir -p repo/{x86_64,any}

#### 3.1. PKGBUILD

Create a PKGBUILD file for your package; View archwiki [creating packages](https://wiki.archlinux.org/title/Creating_packages).

        # Maintainer: Your Name <your.email@example.com>

        pkgname=TxtEd
        pkgver=1.0.0
        pkgrel=1
        pkgdesc="My package"
        arch=('any')
        url="https://github.com/[USERNAME]/TxtEd"
        license=('GPL')
        depends=('python' 'python-pyqt6' 'python-pyqt6-webengine' 'python-pygments')
        source=("$pkgname-$pkgver.tar.gz")
        md5sums=('SKIP')

        build() {
        cd "$srcdir/TxtEd-$pkgver"
        # Add your build commands here, e.g., make
        }

        package() {
        cd "$srcdir/TxtEd-$pkgver"
        install -Dm755 txted-qt6.py "$pkgdir/usr/bin/txted"
        install -Dm644 txted-qt6.desktop "$pkgdir/usr/share/applications/mypkg.desktop"
        install -Dm644 txted.png "$pkgdir/usr/share/pixmaps/txted.png"
        }

In the **source=** part, put:

                source=("$pkgname-$pkgver.tar.gz")

You need to do this and also **create the tarball** on the initial package build. If pointed at the repository server, it will give an error because theres nothing there... Later add:

        source=("$pkgname-$pkgver.tar.gz::https://github.com/[USERNAME]/[REPOSITORY]/raw/main/repo/x86_64/mypkg-$pkgver-1-any.pkg.tar.zst")

### 4. Tarball

You've got your chosen repository prepared, created the PKGBUILD, now create the package tarball:

        tar -czvf TxtEd-1.0.0.tar.gz TxtEd

Make the package; this will give you:  **.tar.gz** and **pkg.tar.zst**.

                makepkpg -si

Move the built package tarballs to the appropriate directories, e.g. */repo/x86_64/ /repo/any*. The package tarball will have **.pkg.tar.zst** in the name.

                mv ../TxtEd/TxtEd-1.0.0-1-any.pkg.tar.zst repo/x86_64/

#### 4.1. Gen DB

Generate the package database files and add the **package**: *.pkg.tar.zst to the **repository database**: *.db.tar.gz:

                repo-add repo/x86_64/ArchPkg.db.tar.gz repo/x86_64/TxtEd-1.0.0-1-any.pkg.tar.zst

If you got multiple files just do:

                repo-add ArchPkg.db.tar.gz *.pkg.tar.zst

The repository should now have the following structure:

                repo/
                ├── x86_64/
                │   ├── TxtEd-1.0.0-1-x86_64.pkg.tar.zst
                │   ├── ArchPkg.db.tar.gz
                │   ├── ArchPkg.files.tar.gz

For pacman to see the new db you need to rename the generated databases from tarballs to *.db and *.files files:

                mv ArchPkg.db.tar.gz ArchPkg.db

                mv ArchPkg.files.tar.gz ArchPkg.files

The repository should now look like below and should now be ready to be used.

                ├── repo/
                │       └── x86_64/
                │       ├── ArchPkg.db
                │       ├── ArchPkg.files
                │       ├── TxtEd-1.0.0-1-any.pkg.tar.zst

#### 4.1.1 Sign DB

Sign the update db files:

                gpg --detach-sign --use-agent --armor --output ArchPkg.db.tar.gz.sig ArchPkg.db.tar.gz

### 2.4. Update

Make sure to update the pacman database:

                sudo pacman -Sy
