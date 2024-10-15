+++
title = 'Building a Cloud File Uploader with Bash (Phase 1 of Learn to Cloud)'
date = 2024-10-15T10:10:00+02:00
slug = "building-bash-cloud-file-uploader"
+++

![s3Upload Logo](/s3upload-logo.png)

As part of my learning journey, I decided to get hands-on with a practical project—creating a script that automates file uploads from my local system to Amazon S3 using Bash. This project is a straightforward way to get familiar with cloud concepts while working directly with AWS services. Plus, it’s a great chance to improve my Bash scripting skills.

In this blog post, I’ll take you through the script I built for uploading multiple files to S3, step by step. If you're following along with the [Learn to Cloud guide](https://learntocloud.guide/), this project fits right in with the skills you're picking up.

## Script overview
---

The script I created does a few important things: First, it checks to make sure you’re using Bash 4.0 or higher because older versions might not support some features. Next, it pulls AWS credentials from a `.env` file (which you’ll need to set up) and exports them so the AWS CLI can use them. The script also lets you specify multiple files for upload and create a folder in your S3 bucket if you want to keep things organized.

## Checking the Bash version
---

The script starts by checking that you’re using the right version of Bash:

```
#!/usr/bin/env bash

if [ "${BASH_VERSION%%.*}" -lt 4 ]; then
    echo "This script requires Bash version 4.0 or higher."
    exit 1
fi
```

## Setting up AWS credentials
---

Next, the script gets the AWS credentials from the local `.env` file. This way, one don’t have to worry about putting your keys directly in the script, making it more secure and easier to manage.

```
source .env

export AWS_ACCESS_KEY_ID
export AWS_SECRET_ACCESS_KEY
export AWS_DEFAULT_REGION
```

I also grab the name of the S3 bucket from the .env file:

```
bucket_name=$BUCKET_NAME
```

## Adding some ASCII art
---
To give the script a little character, I included an ASCII art that pops up when you run it. It’s a fun way to make the experience a bit more enjoyable.

```
echo -e "\033[32m
                                                                               
              ████████                       ████                         █████
             ███░░░░███                     ░░███                        ░░███ 
      █████ ░░░    ░███ █████ ████ ████████  ░███   ██████   ██████    ███████ 
     ███░░     ██████░ ░░███ ░███ ░░███░░███ ░███  ███░░███ ░░░░░███  ███░░███ 
    ░░█████   ░░░░░░███ ░███ ░███  ░███ ░███ ░███ ░███ ░███  ███████ ░███ ░███ 
     ░░░░███ ███   ░███ ░███ ░███  ░███ ░███ ░███ ░███ ░███ ███░░███ ░███ ░███ 
     ██████ ░░████████  ░░████████ ░███████  █████░░██████ ░░████████░░████████
    ░░░░░░   ░░░░░░░░    ░░░░░░░░  ░███░░░  ░░░░░  ░░░░░░   ░░░░░░░░  ░░░░░░░░ 
                                   ░███                                     
           https://ognard.com      █████   AWS S3 Multi-File Uploader v1.0           
                                  ░░░░░                                              
\033[0m"
```

> The ASCII art in the script begins with `\033[32m` and ends with `\033[0m`. The `\033[32m` code sets the text color to green. To learn more about using colors in the terminal, check out [this link](https://www.codeproject.com/Articles/5329247/How-to-Change-Text-Color-in-a-Linux-Terminal). Additionally, remember to use the `-e` flag with the echo command to enable the interpretation of backslash escapes.

## Help menu
---
To make the script user-friendly, I've added a little help menu function (that will be called later). In general, if you run the script with the -h flag, it will show you the options available to you.

```
print_help() {
    echo -e "\033[32m
    * ------------------------------------------------------------------------ *

         Options:
                            
            -f     Provide files to upload. Multiple files must be placed
                   in double quotes.

            -d     (Optional) Provide directory name that will be created. 
                   Files will be uploaded in the created directory.

            -h     This screen.

    * ------------------------------------------------------------------------ *

         Example: 

            ./uploader.sh -f \"file1.txt file2.jpg\" -d \"folder_name\"

\033[0m"
}
```

## Uploading files to S3
---
The main function of the script is uploading files. It checks if the file exists locally, builds the S3 path (with or without a directory), and uses the AWS CLI to upload the file.

```
upload_file() {
    local file_name="$1"
    local directory="$2"

     if [[ -f "$file_name" ]]; then
         if [[ -n "$directory" ]]; then
             s3_path="s3://$bucket_name/$directory/$(basename $file_name)"
         else
             s3_path="s3://$bucket_name/$(basename $file_name)"
         fi

         error_message=$(aws s3 cp "$file_name" "$s3_path" --quiet 2>&1)

         if [[ $? -eq 0 ]]; then
             echo -e ">>> \033[32m[SUCCESS]\033[0m $file_name was uploaded successfully to $s3_path"
         else
             echo -e ">>> \033[31m[ERROR]\033[0m File upload failed!\n\t\t$error_message"
         fi
     else
         echo -e ">>> \033[31m[ERROR]\033[0m File $file_name was not found!"
    fi
}
```
1. File Existence Check: The script first checks if the specified file exists. If it doesn't, it outputs an error message.

2. S3 Path Construction: If the directory is provided, it constructs the path accordingly. Otherwise, it uploads the file to the root of the bucket.

3. File Upload: It then uses the AWS CLI to upload the file and captures any error messages if the upload fails.

If something goes wrong, the script will provide details about the error, helping you identify what to fix. 

Additionally, the output messages use color coding, making it easy to see whether the file upload was successful or if there was an error.

## Creating a folder in S3 (Optional)
---
If you want to organize your uploads, the script can create a folder in S3 for you. This is optional but can be really helpful if you’re dealing with a lot of files.

```
create_folder() {
    local directory="$1"
    aws s3api put-object --bucket "$bucket_name" --key "$directory/" > /dev/null 2>&1

    if [[ $? -eq 0 ]]; then
        echo -e ">>> \033[32m[SUCCESS]\033[0m Folder '$directory' was created successfully."
    else
        echo -e ">>> \033[31m[ERROR]\033[0m Folder creation failed."
    fi
}
```

This function attempts to create a folder in the specified S3 bucket. If successful, it will notify you, otherwise, it will indicate a failure.

## Parsing command-line options
---
I've implemented the `getopts` functionality to parse the provided flags and values from the command line accordingly. If the flag `-h` is specified, it calls the print_help function, which displays the help information. If the `-d` flag (optional) is used, it requires a directory name, specifying where all the provided files will be uploaded. Lastly, if the `-f` flag is provided, it requires the name(s) of the file(s) that need to be uploaded.

```
while getopts 'f:d:h' flag; do
    case "${flag}" in
        h)
            print_help
            exit 0
            ;;
        d)  
            directory="${OPTARG}"
            create_folder "$directory"
            ;;
        f)
            files="${OPTARG}"
            ;;
        *)
echo "
    * ------------------------------------------------------------------------ *

        Unknown flag. Use -h to see the available options.

"
            exit 1
            ;;
    esac
done
```

## Final bit
---

Finally, the script ties everything together. Once you’ve provided the options, it goes through the list of files and uploads them. If you forget to specify any files, it will show the help menu.

```
if [[ -n "$files" ]]; then
    for file_name in $files; do
        upload_file "$file_name" "$directory"
    done
else
    print_help
fi

shift $((OPTIND - 1))
```

This section checks if any files were provided for upload. If files are specified, it loops through each one and calls the `upload_file` function. If no files are specified, it displays the help menu to guide the user.

The shift `$((OPTIND - 1))` command adjusts the positional parameters, allowing any remaining arguments (if any) to be processed further in the script.


## Running the script
---
You can execute the script from your terminal as follows:

```
./uploader.sh -f "file1.txt file2.jpg" -d "folder_name"
```

This command uploads `file1.txt` and `file2.jpg` into the specified `folder_name` directory in your S3 bucket. If you omit the -d option, the files will be uploaded directly to the root of the bucket.


## Conclusion
---

This is the initial version of the script. In future updates, I plan to add more features, such as a progress bar to track the upload process, an option to generate and display a shareable link for the uploaded files, and file synchronization to check if the files or directories already exist in the cloud. You can find the complete script on my [GitHub profile](https://github.com/ognard/s3-upload). 

Thank you for stopping by!