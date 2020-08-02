---
title: "PowerShell parse EML files"
excerpt: "EML are a proprietary, and almost, obsolete used to represent an email message in encoded format which is not easy to parse for automation purposes"
categories:

  - PowerShell
  - HowTo
tags:
  - PowerShell
  - Functions
---

Some time ago I've been faced with a challenge, well a file format I should say, I was not expecting parsing an *EML* file format which, in total honestly, I thought extinct.

To give you some background I implemented a solution sending notifications to a dedicated *Microsoft Teams* channel containing some information that needed to be parsed by a script and eventually take some actions.

I initially thought this would have been rather easy. I was wrong.

## EML an obscure file format

EML file format is defined in [rfc0822](https://www.ietf.org/rfc/rfc0822.txt) with an extension in [rfc1521](https://www.ietf.org/rfc/rfc1521.txt) defining MIME entities but as far as I know there is no official definition of the file format.

Anyhow EML is the format *MS Teams* is using to store email messages that are sent to the a team's address and while retrieving the EML file itself is rather easy extracting any meaningful data from it proved another story.

## EML as a text file

My requirement was to extract the **subject** line of the message as it contained information I needed and, as EML is just like a regular text file, I initially thought I could easily get away with something similar the following:

```powershell
# Define regex
[string]$subjectRegex = '^subject:'

# Get eml content
$emlFile = Get-Content -Path $pathToEmlFile

# Get only relevant data
$emlFile = $emlFile -match $subjectRegex
```

The above was working while tasting the solution in an interactive session but when I started feeding real production data to the script the parsed data was missing key data I knew was in the original eml file (if you're working the subject line contained a *[DO NOT MODIFY]* tag followed by two listing email addresses, so a rather long string).

Issue was being caused by character limit in the subject line which was being broken down to a new line making my filter useless.

## Python to the rescue! Or not?

Not being able to find an easy way out of my *issue* via PowerShell I initially thought using Python to parse the eml file as there is nice [parser](https://pypi.org/project/eml-parser/) available which does exactly what I was looking for.

While working this solution was introducing some other challenges and making the whole solution more complicated then I remembered a colleague dealing with this format in C# back at the time so I started scouting Stack Overflow posts [and found this](https://stackoverflow.com/questions/936422/recommendations-on-parsing-eml-files-in-c-sharp/2838544#2838544) which was describing exactly my problem!

### Convert-EmlFile function

**Convert-EmlFile** is a really simple function which does exactly what is described in the above post, it uses the *ADODB class* to return an object you can easily interact with within PowerShell.

Here's the code:

```powershell
function Convert-EmlFile
{
<#
    .SYNOPSIS
        Function will parse an eml files.

    .DESCRIPTION
        Function will parse eml file and return a normalized object that can be used to extract infromation from the encoded file.

    .PARAMETER EmlFileName
        A string representing the eml file to parse.

    .EXAMPLE
        PS C:\> Convert-EmlFile -EmlFileName 'C:\Test\test.eml'

    .OUTPUTS
        System.Object
#>
    [CmdletBinding()]
    [OutputType([object])]
    param
    (
        [Parameter(Mandatory = $true)]
        [ValidateNotNullOrEmpty()]
        [string]
        $EmlFileName
    )

    # Instantiate new ADODB Stream object
    $adoStream = New-Object -ComObject 'ADODB.Stream'

    # Open stream
    $adoStream.Open()

    # Load file
    $adoStream.LoadFromFile($EmlFileName)

    # Instantiate new CDO Message Object
    $cdoMessageObject = New-Object -ComObject 'CDO.Message'

    # Open object and pass stream
    $cdoMessageObject.DataSource.OpenObject($adoStream, '_Stream')

    return $cdoMessageObject
}
```

The function accepts only a single parameter *-EmlFileName* which is a string representing the path, either local or UNC, to the eml file itself, and will return a **formatted** object you can work with.

**Note:** You will need to supply a valid path, either local or UNC, or function will fail this is not related to PowerShell code but to the ADODB.Stream class requiring a path rather than just the file name.{: .notice--success}

Here's a sample run and output where I've removed sensitive information:

```powershell
# Define path to the eml file
[string]$emlFile = '\\Server01\Data\Message.eml'

Convert-EmlFile -EmlFileName $emlFile

# Output
BCC                  :
CC                   :
FollowUpTo           :
From                 : "Test recipient" <test@pscustomobject.om>
Keywords             :
MimeFormatted        : True
Newsgroups           :
Organization         :
ReceivedTime         : 2/7/2020 1:56:34 PM
ReplyTo              :
DSNOptions           : 0
SentOn               : 2/7/2020 1:56:30 PM
Subject              : [Test Subject] - A very long subject line for the test 
To                   : "PsCustomObject Test recipient" <test@pscustomobject.com>
TextBody             :
                       Mail text + HTML signature if present

HTMLBody             : <html xmlns:v="urn:schemas-microsoft-com:vml" xmlns:o="urn:schemas-microsoft-com:office:office"
                       xmlns:w="urn:schemas-microsoft-com:office:word"
                       xmlns:m="http://schemas.microsoft.com/office/2004/12/omml"
                       xmlns="http://www.w3.org/TR/REC-html40">
<snip>
Attachments          : System.__ComObject
Sender               :
Configuration        : System.__ComObject
AutoGenerateTextBody : False
EnvelopeFields       :
TextBodyPart         : System.__ComObject
HTMLBodyPart         : System.__ComObject
BodyPart             : System.__ComObject
DataSource           : System.__ComObject
Fields               : System.__ComObject
MDNRequested         : False
```

As you can see function will parse the file and output an object that is easy to work with for example to extract the subject line or any other part of the text available in the message.

### Function availability

Function is available in my GitHub repository for functions [here](https://github.com/PsCustomObject/PowerShell-Functions) and will soon be made available in my [IT-ToolBox](https://github.com/PsCustomObject/IT-ToolBox) module as well.

If you find any issue or have any question do not hesitate to to leave me a comment here or open an issue in GitHub.
