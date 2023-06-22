#!/bin/bash

ARGS="$@" # Save arguments, shift will modify $@

usage() {
    echo "tmpdir [options] <directory>"
    echo "============================"
    echo "<directory> - Path to the temporary directory to create"
    echo "========Options============="
    echo "-v / --verbose - Be extra verbose"
    echo "-h / --help - Display this help message"
    echo "-c / --clear - Clear all temporary directories, create nothing"
    echo "-l / --list - List all temporary directories, create nothing"
    echo "-f / --file FILE - Path to the persistent data file, defaults to /etc/tmpdir.conf"
    echo "-t / --type TYPE - Type to mount on the directly (tmpfs, ramfs, etc), defaults to tmpfs"
    echo "-s / --size SIZE - Size of the directory to create, defaults to 1G"
    echo "-m / --mode MODE - Mode to mount the directory with, defaults to 1777"
    echo "============================"
    echo "Please note that this tool does not perform option unpacking, for instance:"
    echo "-vc will not work, you must use -v -c"
}

fs_type() {
    DF=$(df -T "$1" | tail -n 1)
    echo "$DF" | awk '{print $2}'
}

check_priv() {
    if [ "$EUID" -ne 0 ]; then
        # Attempt to elevate privileges if we are not root
        # Attempt pkexec, sudo, su
        if [ -f "$(which sudo)" ]; then
            sudo "$0" $ARGS
            exit $?
        elif [ -f "$(which pkexec)" ]; then
            pkexec "$0" $ARGS
            exit $?
        elif [ -f "$(which su)" ]; then
            su -c "$0" $ARGS
            exit $?
        else
            echo "ERROR: Script must be run as root!"
            exit 1
        fi
    fi
    verbose "Root check success"
}

verbose() {
    if [ "$VERBOSE" -eq 1 ]; then
        echo "[verbose] $1"
    fi
}

get_data() {
    if [ -f "$FILE" ]; then
        # Check that we have read access to the file
        if [ ! -r "$FILE" ]; then
            echo "ERROR: Cannot read $FILE"
            exit 1
        fi
        # Place every line in the file into an array
        IFS=$'\n' read -d '' -r -a lines < "$FILE"
        # Remove every line that starts with a # or that is empty
        lines=($(for line in "${lines[@]}"; do echo "$line"; done | grep -v "^#" | grep -v "^$"))
        # Return the array
        echo "${lines[@]}"
    else
        # The file does not exist, it will be written by the program
        echo ""
    fi
}

create_file() {
    if [ -f "${FILE}" ]; then
        verbose "${FILE} already exists, reading from it"
    else
        verbose "${FILE} does not exist, creating it"
        mkdir -p "$(dirname "${FILE}")"
        touch "${FILE}"
    fi
}

# Parse the command line arguments
while [ "$1" != "" ]; do
    case $1 in
        -v | --verbose )        VERBOSE=1
                                ;;
        -h | --help )           usage
                                exit
                                ;;
        -c | --clear )          CLEAR=1
                                ;;
        -l | --list )           LIST=1
                                ;;
        -f | --file )           shift
                                FILE=$1
                                ;;
        -t | --type )           shift
                                TYPE=$1
                                ;;
        -s | --size )           shift
                                SIZE=$1
                                ;;
        -m | --mode )           shift
                                MODE=$1
                                ;;
        * )                     DIRECTORY=$1
                                ;;
    esac
    shift
done

verbose "Full command line: $0 ${ARGS[@]}"

# Set defaults
if [ -z "$FILE" ]; then
    FILE="/etc/tmpdir.conf"
fi
if [ -z "$TYPE" ]; then
    TYPE="tmpfs"
fi
if [ -z "$SIZE" ]; then
    SIZE="1G"
fi
if [ -z "$MODE" ]; then
    MODE="1777"
fi
if [ -z "$VERBOSE" ]; then
    VERBOSE=0
fi
if [ -z "$CLEAR" ]; then
    CLEAR=0
fi
if [ -z "$LIST" ]; then
    LIST=0
fi

# Check that we are root
check_priv

# DEBUG: Print out the variables
verbose "VERBOSE=$VERBOSE"
verbose "CLEAR=$CLEAR"
verbose "LIST=$LIST"
verbose "FILE=$FILE"
verbose "TYPE=$TYPE"
verbose "SIZE=$SIZE"
verbose "MODE=$MODE"
verbose "DIRECTORY=$DIRECTORY"

create_file

# Get the data from the file
lines=($(get_data))

# Check if we should list the directories
if [ "$LIST" -eq 1 ]; then
    echo "Currently existing temporary directories:"
    # List the directories
    for line in "${lines[@]}"; do
        echo -n "$line - "
        if [ -d "$line" ]; then
            echo -n "Exists, "
            if [ "$(fs_type $line)" == "$TYPE" ]; then
                echo "mounted as $TYPE"
            else
                echo "mounted as $(fs_type $line) (will be pruned on clear)"
            fi
        else
            echo "Does not exist (will be pruned on clear)"
        fi
    done
    exit
fi

# Check if we should clear the directories
if [ "$CLEAR" -eq 1 ]; then
    echo "Clearing all temporary directories..."
    # Loop through the directories
    for line in "${lines[@]}"; do
        # Check if the directory exists
        if [ -d "$line" ]; then
            # Check if the directory is mounted as the correct type
            if [ "$(fs_type $line)" == "$TYPE" ]; then
                # Unmount the directory
                verbose "Unmounting $line"
                umount "$line"
                # The directory is expected to be empty at this point
                verbose "Removing $line"
                rmdir "$line"
            else
                # The directory was unmounded, rmdir it to prevent it from being deleted if it contains files
                verbose "Attempting to remove $line"
                rmdir "$line" 2>/dev/null
            fi
        fi
    done
    # Clear the file
    verbose "Clearing $FILE"
    echo "" > "$FILE"
    exit
fi

# Check if we have a directory to create
if [ -z "$DIRECTORY" ]; then
    echo "ERROR: No directory specified"
    usage
    exit 1
fi

# Normalize the directory path
DIRECTORY=$(realpath "$DIRECTORY")

# Check if the directory already exists
if [ -d "$DIRECTORY" ]; then
    echo "ERROR: $DIRECTORY already exists"
    exit 1
fi

# Check if the directory is already in the file
if [[ " ${lines[@]} " =~ " ${DIRECTORY} " ]]; then
    echo "WARNING: $DIRECTORY is already in the storage file. This is fine, but may indicate a problem."
fi

# Create the directory
verbose "Creating $DIRECTORY"
mkdir -p "$DIRECTORY"

# Mount the directory
verbose "Mounting $DIRECTORY as $TYPE with size $SIZE and mode $MODE"

mount -t "$TYPE" -o "size=$SIZE,mode=$MODE" "$TYPE" "$DIRECTORY" > /tmp/tmpdir.log

# Check if the mount was successful
if [ $? -ne 0 ]; then
    echo "ERROR: Failed to mount $DIRECTORY as $TYPE with size $SIZE and mode $MODE with error code $? (check validity of arguments)"
    echo "ERROR: $(cat /tmp/tmpdir.log)"
    exit 1
fi

# Add the directory to the file
verbose "Adding $DIRECTORY to $FILE"
echo "$DIRECTORY" >> "$FILE"

# Cleanup
rm /tmp/tmpdir.log

# Print out the directory
echo "$DIRECTORY"