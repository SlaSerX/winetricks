
#--------------------------------------------------------------------
#
# Winetricks is a package manager for win32 dlls and applications on posix.
# Features:
# - Consists of a single shell script - no installation required
# - Downloads packages automatically from original trusted sources
# - Points out and works around known wine bugs automatically
# - Both commandline and GUI operation
# - Can install many packages in silent (unattended) mode
# - Multiplatform; written for Linux, but supports MacOSX and Cygwin, too
#
# Uses the following non-Posix system tools:
# - wine is used to execute win32 apps except on cygwin.
# - cabextract, unrar, unzip, and 7z are needed by some verbs.
# - aria2c, wget, or curl is needed for downloading.
# - sha1sum or openssl is needed for verifying downloads.
# - zenity is needed by the GUI, though it can limp along somewhat with kdialog.
# - xdg-open (if present) or open (for Mac OSX) is used to open download pages
#   for the user when downloads cannot be fully automated.
# - sudo is used to mount .iso images if the user cached them with -k option.
# - perl is used to munge steam config files
# On ubuntu, the following lines can be used to install all the prereqs:
#    sudo add-apt-repository ppa:ubuntu-wine/ppa
#    sudo apt-get update
#    sudo apt-get install cabextract p7zip unrar unzip wget wine1.7 zenity
#
#--------------------------------------------------------------------
#
# Copyright
#   Copyright (C) 2007-2014 Dan Kegel <dank!kegel.com>
#   Copyright (C) 2008-2015 Austin English <austinenglish!gmail.com>
#   Copyright (C) 2010-2011 Phil Blankenship <phillip.e.blankenship!gmail.com>
#   Copyright (C) 2010-2015 Shannon VanWagner <shannon.vanwagner!gmail.com>
#   Copyright (C) 2010 Belhorma Bendebiche <amro256!gmail.com>
#   Copyright (C) 2010 Eleazar Galano <eg.galano!gmail.com>
#   Copyright (C) 2010 Travis Athougies <iammisc!gmail.com>
#   Copyright (C) 2010 Andrew Nguyen
#   Copyright (C) 2010 Detlef Riekenberg
#   Copyright (C) 2010 Maarten Lankhorst
#   Copyright (C) 2010 Rico Schüller
#   Copyright (C) 2011 Scott Jackson <sjackson2!gmx.com>
#   Copyright (C) 2011 Trevor Johnson
#   Copyright (C) 2011 Franco Junio
#   Copyright (C) 2011 Craig Sanders
#   Copyright (C) 2011 Matthew Bauer <mjbauer95>
#   Copyright (C) 2011 Giuseppe Dia
#   Copyright (C) 2011 Łukasz Wojniłowicz
#   Copyright (C) 2011 Matthew Bozarth
#   Copyright (C) 2013-2014 Andrey Gusev <andrey.goosev!gmail.com>
#   Copyright (C) 2013-2015 Hillwood Yang <hillwood!opensuse.org>
#
# License
#   This program is free software; you can redistribute it and/or
#   modify it under the terms of the GNU Lesser General Public
#   License as published by the Free Software Foundation; either
#   version 2 of the License, or (at your option) any later
#   version.
#   This program is distributed in the hope that it will be useful,
#   but WITHOUT ANY WARRANTY; without even the implied warranty of
#   MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
#   GNU Lesser General Public License for more details.
#   You should have received a copy of the GNU Lesser General Public
#   along with this program.  If not, see <http://www.gnu.org/licenses/>.
#
#--------------------------------------------------------------------
# Coding standards:
#
# Portability:
# - Portability matters, as this script is run on many operating systems
# - No bash, zsh, or csh extensions; only use features from
#   the Posix standard shell and utilities; see
#   http://pubs.opengroup.org/onlinepubs/009695399/utilities/xcu_chap02.html
# - 'checkbashisms -p -x winetricks' should show no warnings (per Debian policy)
# - Prefer classic sh idioms as described in e.g.
#   "Portable Shell Programming" by Bruce Blinn, ISBN: 0-13-451494-7
# - If there is no universally available program for a needed function,
#   support the two most frequently available programs.
#   e.g. fall back to wget if curl is not available; likewise, support
#   both sha1sum and openssl.
# - When using unix commands like cp, put options before filenames so it will
#   work on systems like MacOSX.  e.g. "rm -f foo.dat", not "rm foo.dat -f"
#
# Formatting:
# - Your terminal and editor must be configured for utf-8
#   If you do not see an o with two dots over it here [ö], stop!
# - Do not use tabs in this file or any verbs.
# - Indent 4 spaces.
# - Try to keep line length below 80 (makes printing easier)
# - Open curly braces ('{') and 'then' at beginning of line,
#   close curlies ('}') and 'fi' should line up with the matching { or if,
#   cases aligned with 'case' and 'esac'.  For instance,
#
#      if test "$FOO" = "bar"
#      then
#         echo "FOO is bar"
#      fi
#      case "$FOO" of
#      bar) echo "FOO is still bar" ;;
#      esac
#
# Commenting:
# - Comments should explain intent in English
# - Keep functions short and well named to reduce need for comments
#
# Naming:
# Public things defined by this script, for use by verbs:
# - Variables have uppercase names starting with W_
# - Functions have lowercase names starting with w_
#
# Private things internal to this script, not for use by verbs:
# - Local variables have lowercase names starting with uppercase _W_
# - Global variables have uppercase names starting with WINETRICKS_
# - Functions have lowercase names starting with winetricks_
# FIXME: A few verbs still use winetricks-private functions or variables.
#
# Internationalization / localization:
# - Important or frequently used message should be internationalized
#   so translations can be easily added.  For example:
#     case $LANG in
#     de*) echo "Das ist die deutsche Meldung" ;;
#     *)   echo "This is the English message" ;;
#     esac
#
#--------------------------------------------------------------------
