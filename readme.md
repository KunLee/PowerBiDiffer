![.NET Core](https://github.com/rajrao/PowerBiDiffer/workflows/.NET%20Core/badge.svg)

## Introduction ##

This tool can be used to extract and display the power query mashup formulas in use by your PBIX file. Its primary purpose is to work with #GIT, but can be used standalone to. There are 2 ways in which it can be used with GIT: using GIT's differencing engine using **textconv** driver and using an external tool (such as visual studio or WinMerge). In addition, the tool also can be used with JSON files. This allows you to difference dataflow JSON files that you might have exported out of PowerBi online.

**This tool does not allow you to perform merges and using it for merges will likely cause corruption of your files.** Its intended usage it to perform comparison between different versions to make sure the changes you are checking into source control are what you intended to apply. It currently does not have the ability to compare measures and columns added to the datamodel via DAX.

## PBIX ##

When used with PBIX, the tool extracts the mashup formulas in your PBIX file. If used with the textconv driver, the tool extracts the mashup and outputs it to standard output (console). This is done, because GIT uses the std-output and extracts the text and performs its differencing using its enternal diff engine. This allows you to use the [*git diff*](https://git-scm.com/docs/git-diff) command with its various options. When used with an external tool like VisualStudio, the tool extracts the mashup from the 2 input files and saves them to temp files, the temp files are provided to the external tool to perform its differencing.

# Instructions #


### Download the tool ###

1. Get the latest set of files and download it to your computer from [releases](https://github.com/rajrao/PowerBiDiffer/releases)
	
	a. Expand the assets section under the latest release and download the PowerBiDiffer.zip file.
2. Unzip the contents of the zip file to a folder on your computer (tip: before doing this, right click on the zip file and go to properties and check the "Unblock" checkbox and click ok).

### Update GIT Configuration ###

3. Run the script **"pbiDiffer_installInGit.cmd"**. This script must be run from the location where the PowerBiDiffer tool has been downloaded. 
*The tool can be uninstalled from GIT using: "pbiDiffer_uninstallInGit.cmd"*

### Performing Comparisions ###

#### Using Visual Studio ####

You can right click on a file and select the compare with previous version option. Similarly, you can view the history of your file and perform comparsions between versions

#### Using Other tools ####

You can use this tool with VsCode and WinMerge. Uncomment the appropriate sections in the script PowerBiDiffer.cmd, to use it with your favorite tool.

#### Advanced Users: Using it from the Git command prompt: ####

These are instructions for using the tool with git from the command prompt.

	git difftool *.pbix

	git difftool *.json

Compare using PowerBiDiffer the commit that was 3 revisions prior

	git difftool HEAD~3 -t PowerBiDiffer

Display the difference for only files with extension \*.json, comparing current set of files to previous check in (useful when you are looking at changes in branch)
	
	git difftool HEAD^^ *.json

Display the difference between current commit and the previous commit (use ^ to signify how far back)
	
	git difftool HEAD^ HEAD
	
Display the difference between 2 commits (all of these commands can be used with just *git diff* too).
	
	git difftool 3208 fc4b
		
Display the difference between commit 3208 and HEAD
	
	git difftool 3208 HEAD
		
Display the difference for only files with extension \*.json
	
	git difftool *.json
		
	
#### Uses the command line and Gits diff tool ####
	
The following steps need to be done only if you wish to use the gitdiff tool
	
1. Add a .gitattributes file if you dont already have one (this is located at the root of your repo)
	
1. Add the following text to the .gitattributes file (these are needed if you wish to use git diff, but not git difftool)

```
*.PBIX   diff=pbix
*.pbix   diff=pbix
*.json	 diff=json
*.JSON	 diff=json
```

### Commands ###

	git diff *.pbix
	
	or
	
	git diff *.json
		
*to exit out of the diff mode hit q*

	
## Standalone Usage ##

1. The tool can be run on its own and has 2 modes: textconv and difftool.

1. Textconv takes a single file as input and spits out to std output the text.

1. The difftool takes 2 files as input and converts them to text and invokes the diff tool specified on the command line to run.

## Examples ##

1. spit out the json unminified and \r\n replaced with newlines

		textconv "testA.json"

2. Use a tool to display the differences

	### Visual Studio ###

		difftool "testA.json" "testb.json" -d "C:\Program Files (x86)\Microsoft Visual Studio\2019\Enterprise\Common7\IDE\devenv.exe" -a "/diff \"{lp}\" \"{rp}\" \"{ln}\" \"{rn}\""

	### WinMerge ###

		difftool "testA.json" "testb.json" -d "C:\Program Files (x86)\WinMerge\winmergeu.exe" -a "/xq /e /s /dl \"{ln}\" /dr \"{rn}\" \"{lp}\" \"{rp}\"" -v


## Command Line Options ##

### Verbs: ###

```

textconv    Mode: Text Conversion
difftool    Mode: Diff Tool
help        Display more information on a specific command.
version     Display version information.

```
### Options: ###

```

-d, --difftool    Diff Tool Path
-a, --args        Diff Tool arguments. Supported templates: {localFilePath} or
                  {lp}: Local file path{remoteFilePath} or {rp}: Remote file
                  path{localFileName} or {ln}: local file name (for
                  description){remoteFileName} or {rn}: remote file name (for
                  description)
-v                (Default: false) Verbose
-l                (Default: false) Break into debugger
-j                (Default: true) Treat json files as JSON and not as text value 
pos. 0 Required.  Local file value 
pos. 1            Remote file
--help            Display this help screen.
--version         Display version information.

```


### Manually fixing GIT configuration ###

These steps shouldnt have to be performed and are needed only if * Update GIT Configuration * did not work.

1. Open the config file in the .git folder of your repo

1. Add the following code: *(Make sure you update the location to where you have put PowerBiDiffer. Also update path to Visual Studio. if you instead want to use WinMerge, replace commands appropriates)*

```
[diff]
	tool = PowerBiDiffer
	guitool = PowerBiDiffer
[difftool]
	prompt = false
[difftool "PowerBiDiffer"]
	cmd = \"c:\\PowerBiDiffer\\PowerBiDiffer.exe\" difftool "$LOCAL" "$REMOTE" -d \"C:\\Program Files (x86)\\Microsoft Visual Studio\\2019\\Enterprise\\Common7\\IDE\\devenv.exe\" -a '/diff \"{lp}\" \"{rp}\" \"{ln}\" \"{rn}\"'
	#uncomment below to use winmerge
	#cmd = \"c:\\PowerBiDiffer\\PowerBiDiffer.exe\" difftool "$LOCAL" "$REMOTE" -d \"C:\\Program Files (x86)\\WinMerge\\winmergeu.exe\" -a '/xq /e /s /dl \"{ln}\" /dr \"{rn}\" \"{lp}\" \"{rp}\"'
	keepBackup = false
[diff "json"]
	textconv = \"c:\\PowerBiDiffer\\PowerBiDiffer.exe\" textconv
[diff "pbix"]
	textconv = \"c:\\PowerBiDiffer\\PowerBiDiffer.exe\" textconv
```
1. Save the file
