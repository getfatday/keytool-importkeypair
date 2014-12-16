#! /bin/bash
#
# This file is part of keytool-importkeypair.
#
# keytool-importkeypair is free software: you can redistribute it
# and/or modify it under the terms of the GNU General Public License
# as published by the Free Software Foundation, either version 3 of
# the License, or (at your option) any later version.
#
# keytool-importkeypair is distributed in the hope that it will be
# useful, but WITHOUT ANY WARRANTY; without even the implied warranty
# of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU
# General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with keytool-importkeypair.  If not, see
# <http://www.gnu.org/licenses/>.
#

DEFAULT_KEYSTORE=$HOME/.keystore
keystore=$DEFAULT_KEYSTORE
pk8=""
cert=""
alias=""
passphrase=""
tmpdir=""

scriptname=`basename $0`

usage() {
cat << EOF
usage: ${scriptname} [-k keystore] [-p storepass]
-pk8 pk8 -cert cert -alias key_alias

This script is used to import a key/certificate pair
into a Java keystore.

If a keystore is not specified then the key pair is imported into
~/.keystore in the user's home directory.

The passphrase can also be read from stdin.
EOF
}

cleanup() {
if [ ! -z "${tmpdir}" -a -d ${tmpdir} ]; then
   rm -fr ${tmpdir}
fi
}

while [ $# -gt 0 ]; do
        case $1
        in
                -p | --passphrase | -passphrase)
                        passphrase=$2
                        shift 2
        ;;
                -h | --help)
                        usage
                        exit 0
        ;;
                -k | -keystore | --keystore)
                        keystore=$2
                        shift 2
        ;;
                -pk8 | --pk8 | -key | --key)
                        pk8=$2
                        shift 2
        ;;
                -cert | --cert | -pem | --pem)
                        cert=$2
                        shift 2
        ;;
                -a | -alias | --alias)
                        alias=$2
                        shift 2
        ;;
                *)
                        echo "${scriptname}: Unknown option $1, exiting" 1>&2
                        usage
                        exit 1
        ;;
        esac
done

if [ -z "${pk8}" -o -z "${cert}" -o -z "${alias}" ]; then
   echo "${scriptname}: Missing option, exiting..." 1>&2
   usage
   exit 1
fi


for f in "${pk8}" "${cert}"; do
    if [ ! -f "$f" ]; then
       echo "${scriptname}: Can't find file $f, exiting..." 1>&2
       exit 1
    fi
done

if [ ! -f "${keystore}" ]; then
   storedir=`dirname "${keystore}"`
   if [ ! -d "${storedir}" -o ! -w "${storedir}" ]; then
      echo "${scriptname}: Can't access ${storedir}, exiting..." 1>&2
      exit 1
   fi
fi

# Create temp directory ofr key and pkcs12 bundle
tmpdir=`mktemp -q -d "/tmp/${scriptname}.XXXX"`

if [ $? -ne 0 ]; then
   echo "${scriptname}: Can't create temp directory, exiting..." 1>&2
   exit 1
fi

key="${tmpdir}/key"
p12="${tmpdir}/p12"

if [ -z "${passphrase}" ]; then
   # Request a passphrase
  read -p "Enter a passphrase: " -s passphrase
  echo ""
fi

# Convert PK8 to PEM KEY
openssl pkcs8 -inform DER -nocrypt -in "${pk8}" -out "${key}"

# Bundle CERT and KEY
openssl pkcs12 -export -in "${cert}" -inkey "${key}" -out "${p12}" -password pass:"${passphrase}" -name "${alias}"

# Print cert
echo -n "Importing \"${alias}\" with "
openssl x509 -noout -fingerprint -in "${cert}"

# Import P12 in Keystore
keytool -importkeystore -deststorepass "${passphrase}" -destkeystore "${keystore}" -srckeystore "${p12}" -srcstoretype PKCS12 -srcstorepass "${passphrase}" 

# Cleanup
cleanup
