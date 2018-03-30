# For more Linux tips and tricks visit: http://linuxexplore.com
# tarcrypt
Linux File encryption/decryption using openssl | password protected tar

Openssl is one of the best tools which can be used to encrypt/decrypt files. You can password protect your important data to avoid misuse.

To encrypt your files use the command:

openssl des3 -salt -in $FILENAME -out ${FILENAME}.des3
To decrypt the file use the command:

openssl des3 -d -salt -in ${FILENAME}.des3 -out ${FILENAME}
You can also use my script ‘tarcrypt.sh’ to encrypt/decrypt files. This script is using tar to compress/decompress with encryption/decryption functionality.

#!/bin/sh
#
# 'tarcrypt.sh' script can used to compress/decompress the data with encryption.
#
# This script is created & tested by Rahul Panwar.
# WARNING!!! Use it at your own risk.
# Please report the bugs or queries to panwar.rahul@gmail.com

VERSION="Version 1.1.0.1\nCreated by: Rahul Panwar"
PASS=""
PASS_OPTION=""
EXT_OPTION=""
COMP_FILE="encrypted_file"

DATA_FILES_ALL=""

# Usage
usage ()
{
	echo "Usage:"
	echo "	${0##*/} -c \" [  ... \"  [-p ]"
	echo "	${0##*/} -x  [-C ] [-p ]"
	echo "	${0##*/} -h"
	echo "	${0##*/} -v"
	echo "OPTIONS:"
	echo "	-c|--compress	: Compress and encrypt the file(s) or directory(ies)
					for multiple files use double quotes (for example \"file1 file2 dir1\")."
	echo "	-x|--decompress	: Decrypt and uncompress the file"
	echo "	-p|--password	: Password to encrypt/decrypt the file"
	echo "	-C|--extract	: Change directory, to extract the compressed file, default is current directory"
	echo "	-h|--help	: To see this help"
	echo "	-v|--version	: Check the version"
}
[ $# = 0 ] && usage && exit 1

# to encrypt files using openssl
encrypt_file()
{
	FNAME=$1

	#openssl des3 -salt -in "$FNAME" -out "$FNAME.des3"
	openssl des3 -salt -out "$FNAME" ${PASS_OPTION}
}

# to decrypt the files using openssl
decrypt_file()
{
	FNAME=$1

	#openssl des3 -d -salt -in "$FNAME" -out "${FNAME%.[^.]*}"
	openssl des3 -d -salt -in "$FNAME" ${PASS_OPTION}
}

# compress and encrypt the files
en_comp()
{
	tar -czp ${DATA_FILES} | encrypt_file ${COMP_FILE}
}

# decrypt and uncompress the files
de_comp()
{
	decrypt_file ${COMP_FILE} | tar ${EXT_OPTION} -xz
}

# main function
main_function()
{
	compress=""
	decompress=""
	extract=""
	password=""
	while test "$1" != "" ; do
		OPT=$1
		OPT_VAL1=$2
		OPT_VAL2=$3
		case "$OPT" in
			--compress|-c)
				DATA_FILES_ALL="${OPT_VAL1}"
				[ ! "${OPT_VAL2}" ] && echo -e "No encrypt filename, using default name: ${COMP_FILE}"
				COMP_FILE=${OPT_VAL2:-$COMP_FILE}
				compress=1
				shift
				[ "$OPT_VAL2" ] && shift
			;;
			--decompress|-x)
				COMP_FILE=${OPT_VAL1:-$COMP_FILE}
				decompress=1
				shift
			;;
			--password|-p)
				PASS=${OPT_VAL1}
				[ "$PASS" ] && password=1 && PASS_OPTION="-pass pass:${PASS}"
				shift
			;;
			--extract|-C)
				[ ! -d "$OPT_VAL1" ] && echo -e "Extract directory not exists" && exit 1
				EXT_OPTION="-C ${OPT_VAL1:-$PWD}"
				extract=1
				shift
			;;
			--help|-h)
				usage
				exit 0
			;;
			--version|-v)
				echo -e "${VERSION}"
				exit 0
			;;
			-*)
				echo "Error: no such option $OPT"
				usage
				exit 1
			;;
			*)
				echo -e "Error: invalid option $OPT"
				usage
				exit 1
		esac
		shift
	done

	if [ "$compress" ] && [ "$decompress" ]; then
		echo -e "\n-c and -x can't be used simultaneously\n" && usage && exit 1
	elif [ "$extract" ] && [ "$compress" ]; then
		echo -e "\n-C can only use with -x option\n" && usage && exit 1
	fi
}

main_function "$@"
if [ "$compress" ]; then
	for FILES in ${DATA_FILES_ALL}; do
		[ -e "${FILES}" ] && DATA_FILES="${DATA_FILES} ${FILES}"
	done
	if [ ! "${DATA_FILES}" ]; then
		echo -e "No file(s) found to compress" && exit 2
	fi
	echo "==> compressing"
	en_comp &>/dev/null && echo "success" || echo "failure"
elif [ "$decompress" ]; then
	[ ! -e "${COMP_FILE}" ] && echo -e "No file to decrypt" && exit 2
	echo "==> decompressing"
	de_comp &>/dev/null && echo "success" || echo "failure"
fi

If you found any bug in the script, please write your comment. I like to improve this, so suggestions are most welcome.
