CMD Hacks: Finding and Copying Modified Files Without GIT
=========================================================


![image](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*3dWbNPtG8s2aDMLbod7Dgg@2x.png)

The other day, I needed to make an urgent change on a Windows server and made modifications across multiple folders. Since Git was not installed on the server, I thought it would be difficult to identify which files I had changed. However, after doing some research, I realized that I could quickly solve this issue with a command I had never used before. This command was the: **forfiles**

```
forfiles /P "C:\Project_Folder" /S /D +2 /C "cmd /c echo @path"
```

Explanation of this command:

*   `/P "C:\Project_Folder"`: Specifies the path of the folder you want to scan.
*   `/S`: This tells the command to scan subfolders.
*   `/D +2`: Lists files that have been modified in the last two days. However, if you need to use a specific date, this parameter can be updated as follows: `/D +13.10.2024` (lists files modified after this date).
*   `/C "cmd /c echo @path"`: Displays the full file paths on the screen. `@path` represents the full path of the file.

As we will understand from here, the `forfiles` command actually works like a `forloop`, detecting the files in the criteria we provide and executes a separate command for each.

My goal was to identify the files I modified and export these files to a separate folder while maintaining the folder structure. Then I took these files to my own computer and planned to create a new commit in Git.

I performed this operation by using the `copy` command instead of the `echo` command:

```
forfiles /P "C:\Project_Folder" /S /D +13.09.2024 /C "cmd /c copy @path C:\Users\User\Desktop\Commit"
```

This command copied all files that changed after the specified date to the `commit` folder. However, the folder structure was not preserved when copying files. I applied the `xcopy` command to protect the folder structure:

```
forfiles /P "C:\Project_Folder" /S /D +13.09.2024 /C "cmd /c xcopy @path C:\Users\User\Desktop\Commit@relpath /S /Y"
```

Details of this command:

*   `@path`Specifies the full path of the file.
*   `@relpath`It refers to the file path after the specified scan folder.
*   `/S`Allows copying with subfolders.
*   `/Y` Automatically answers `YES` to the question “Overwrite (Y/N)?”.

However, with the `forfiles` command, not only the files come, folders were also included in the loop into a separate iteration and caused problems with the `xcopy` command. Therefore, since I was only interested in files, the folders had to be disabled. I used the `@isdir` variable for this.

```
forfiles /P "C:\Project_Folder" /S /D +13.09.2024 /C "cmd /c if @isdir==FALSE xcopy @path C:\Users\User\Desktop\WebCommit\Commit\@relpath /S /Y
```

Everything was great except one thing. For each file that came with **forfiles**, xcopy asked us a question in the following way:

```
robots.txt specify a file name or directory name on the target
(F = file, D = directory)?
```

It was inefficient to manually press the “F” key for each file. I created a `txt` file to solve this problem. I just wrote “F” in it and integrated this file with the command.

```
forfiles /P "C:\Project_Folder" /S /D +13.09.2024 /C "cmd /c if @isdir==FALSE xcopy @path C:\Users\User\Desktop\WebCommit\Commit\@relpath /S /Y<C:\Users\User\Desktop\WebCommit\DirOrFolder.txt"
```

In this way, I automatically answered the question the xcopy command asked for each file.

By automating file detection and copying with CMD, I was able to simulate a basic version of Git functionality in a non-Git environment. This approach allows for easy identification and extraction of modified files while preserving the folder structure.

I hope this solution proves useful for your file management needs too!
