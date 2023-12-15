# CHANGELOG

All notable changes to this project will be documented in this file.

The format is based on [Common Changelog](https://common-changelog.org/).

## 1.2.0.33 - 2023-12-15

### Added

- Ready for acceptance testing (Debian 12) release

## 1.1.2.32 - 2023-12-14

### Added

- Final touches for UFW rules and application profiles

## 1.1.2.31 - 2023-12-14

### Added

- Tweak a lot of UFW rules and application profiles

## 1.1.1.30 - 2023-12-14

### Added

- Add outbound UFW Ports for SSSD (Active Directory) and Samba (Fileserver)

## 1.1.1.29 - 2023-12-14

### Added

- Add outbound UFW Ports for Kerberos, Add DNS TCP Port (for Windows DNS)

## 1.1.1.28 - 2023-12-14

### Fixed

- Add Hyper-V workaround scripts (`hv_get_dhcp_info` `hv_get_dns_info` `hv_set_ifconfig`) to stop the spamming of the `hv_kvp_daemon`

## 1.1.1.28 - 2023-12-14

### Changed

- Tweaked the Fail2Ban configuration for Debian 12

## 1.1.1.27 - 2023-12-14

### Added

- **Breaking:** Add Fail2Ban back to the system

## 1.1.1.26 - 2023-12-14

### Changed

- Better spring clean of the system (log files, etc.)

## 1.1.1.26 - 2023-12-14

### Changed

- Prevent parameter expansion by add `'EOF'` to the file creation, where needed

## 1.1.1.26 - 2023-12-13

### Changed

- Add some aliases

## 1.1.1.26 - 2023-12-13

### Changed

- Add some more profile settings

## 1.1.1.25 - 2023-12-13

### Changed

- Add some more security settings

## 1.1.1.24 - 2023-12-13

- Add some security settings

## 1.1.1.24 - 2023-12-13

### Fixed

- Fix issue with .NET handling (profile)

## 1.1.1.23 - 2023-12-13

### Fixed

- Removed some more typos from the config files

## 1.1.1.22 - 2023-12-12

### Fixed

- Squished some bugs (mostly config typos)

## 1.1.1.21 - 2023-12-12

### Changed

- Fixed DHCPv6 issues with UFW

## 1.1.1.20 - 2023-12-12

### Changed

- Make the UFW rules more generic

## 1.1.1.19 - 2023-12-12

### Changed

- Removed some old UFW rules

## 1.1.1.19 - 2023-12-12

### Security

- Tweaked the UFW Rules

## 1.1.1.18 - 2023-12-12

### Deprecated

- **Breaking:** Removed Postfix from the default image

## 1.1.1.18 - 2023-12-12

### Removed

- Removed some old stuff

## 1.1.1.17 - 2023-12-12

### Added

- Add PowerShell 7

## 1.1.1.17 - 2023-12-12

### Added

- Add .NET handling

## 1.1.1.16 - 2023-12-11

### Added

- Add OMI default configuration
- Add DSC default configuration

## 1.1.1.15 - 2023-12-11

### Added

- Add OMI installation
- Add DSC installation

## 1.1.1.14 - 2023-12-11

### Added

- Add the Microsoft repository for PowerShell 7 (not working yet on Debian 12)

## 1.1.1.13 - 2023-12-11

- Adopted the script to Debian 12

## 1.1.1.13 - 2023-12-10

- Changed the CIDR of the management network (AllowPingFromNetwork)

## 1.1.1.12 - and older

_This releases where never published._

[![Common Changelog](https://common-changelog.org/badge.svg)](https://common-changelog.org)