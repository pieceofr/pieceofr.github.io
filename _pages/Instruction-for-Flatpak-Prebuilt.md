If user wants to have pre-built `bitmarkd` binary, there are 2 ways to use, one is from [bitmark-node](https://github.com/bitmark-inc/bitmark-node), and the other one is `flatpak`.

Flatpak is a framework for distributing desktop applications on Linux. It has been created by developers who have a long history of working on the Linux desktop, and is run as an independent open source project. The official website at [here](https://flatpak.org).

In order to use `flatpak` on Linux distribution, please install it first. Following commands are tested on Ubuntu 18.04, other Linux distribution should have similar flow.

# System requirement

  Install `flatpak`

  ```
  apt install flatpak
  flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo --user
  ```

# Installation

  Download flatpak files from s3: [bitmarkd](https://s3-ap-northeast-1.amazonaws.com/bitmarkd-flatpak/bitmarkd.flatpak) and [recorderd](https://s3-ap-northeast-1.amazonaws.com/bitmarkd-flatpak/recorderd.flatpak)

  If there's any previous installed bitmarkd or recorded, please remove it first

  ```
  flatpak uninstall com.bitmark.bitmarkd
  flatpak uninstall com.bitmark.recorderd
  ```

Then use following command to reinstall new one

  ```
  flatpak install --user bitmarkd.flatpak
  flatpak install --user recorderd.flatpak
  ```

# Run program

  Please create directory `bitmarkd-data` and `recorderd-data` at the place to run program. Usually it's at home directory, but it doesn't matter as long as program is executed at that location.

  `bitmard.conf` should be placed at `bitmarkd-data`, and `recorderd.conf` should be placed at `recorderd-data`.

  For first time start program (or any certificate has not been generated), execute program by:

  ```
  flatpak run com.bitmark.bitmarkd run.sh --init
  flatpak run com.bitmark.recorderd run.sh --init
  ```

  If the certificate has been generated, use following commands to execute:

  ```
  flatpak run com.bitmark.bitmarkd run.sh
  flatpak run com.bitmark.recorderd run.sh
  ```