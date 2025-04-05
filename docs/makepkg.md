# Creating a package repository

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

#### Makepkg Key

To build packages signed with this key it needs to be configured in makepkg in */etc/makepkg.conf*

        #-- Packager: name/email of the person or organization building packages
        PACKAGER="[NAME] <EMAIL>"
        #-- Specify a key to use for package signing
        GPGKEY="[GPG-KEY]"

### 2. Repository

Create the repository directories:

        mkdir -p repo/{x86_64,any}

#### 2.1. Build Packages

Create a PKGBUILD file; View archwiki [creating packages](https://wiki.archlinux.org/title/Creating_packages).

        # Maintainer: Your Name <your.email@example.com>

        pkgname=MyPkg
        pkgver=1.0.0
        pkgrel=1
        pkgdesc="My package"
        arch=('any')
        url="https://github.com/[USERNAME]/MyPkg"
        license=('GPL')
        depends=('python' 'python-pyqt6' 'python-pyqt6-webengine' 'python-pygments')
        source=("$pkgname-$pkgver.tar.gz")
        md5sums=('SKIP')

        build() {
        cd "$srcdir/TxtEd-$pkgver"
        # Add your build commands here, e.g., make
        }

        package() {
        cd "$srcdir/MyPkg-$pkgver"
        install -Dm755 txted-qt6.py "$pkgdir/usr/bin/mypkg"
        install -Dm644 txted-qt6.desktop "$pkgdir/usr/share/applications/mypkg.desktop"
        install -Dm644 mypkg.png "$pkgdir/usr/share/pixmaps/mypkg.png"
        }

#### 2.2. Tarball

Create package tarball:

        tar -czvf MyPkg-1.0.0.tar.gz MyPkg

Add the tarball to the repository as a [release](https://docs.github.com/en/repositories/releasing-projects-on-github/about-releases). Tag the new release e.g. v1.0, then upload the relase.

#### 2.3. Make Package

Build the package; this will make two tarballs, **.tar.gz** and **pkg.tar.zst**.

                makepkpg -si

Move the built package tarballs to the appropriate directories, e.g. */repo/x86_64/ /repo/any*. The package tarball will have **pkg.tar.zst** in the name.

                mv ../MyPkg/MyPkg-1.0.0-1-any.pkg.tar.zst repo/x86_64/

### 2.4. Database Files



Then generate the package database files:

                repo-add repo/x86_64/MyPkg.db.tar.gz repo/x86_64/MyPkg-1.0.0-1-any.pkg.tar.zst

The repository should now have the following structure:

        repo/
        ├── x86_64/
        │   ├── your-package-1.0.0-1-x86_64.pkg.tar.zst
        │   ├── your-repo.db.tar.gz
        │   ├── your-repo.files.tar.gz
        ├── any/
        │   ├── your-package-1.0.0-1-any.pkg.tar.zst
        │   ├── your-repo.db.tar.gz
        │   ├── your-repo.files.tar.gz

### 3. Repository to Pacman

Add your repository to pacman in */etc/pacman.conf*. Add *TrustAll* and the repository server.

                [txted-repo]
                SigLevel = Optional TrustAll
                Server = https://[USERNAME].github.io/[REPOSITORY]/repo/x86_64
