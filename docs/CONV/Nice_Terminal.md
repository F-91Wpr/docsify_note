---
data: 2023-01-02
---
# Nice_Terminal

本篇为美化终端的教程，参考官方文档、[Scott](https://www.youtube.com/watch?v=VT2L1SXFq9U&t=1237s) 和 [dveaslife](https://www.youtube.com/watch?v=5-aK2_WwrmM)。

Oh My Posh 是一个定制提示符引擎（custom prompt engine），在5.0版本后支持跨平台，因此本配置方案可以从 Windows 迁移到 Linux。（duck不必）。

系统：Windows11

终端：Windows Terminal

![图 1](https://cdn.jsdelivr.net/gh/Z-404/imageHost@main/2023/01/MI_20230105_1672930174274.png)  

## 安装 Nerd Font

1. 下载字体[CaskaydiaCove NF](https://github.com/ryanoasis/nerd-fonts/releases/download/v2.1.0/CascadiaCode.zip?WT.mc_id=-blog-scottha)

2. 将字体文件直接拖入`C:\Windows\Fonts`

![图 4](https://cdn.jsdelivr.net/gh/Z-404/imageHost@main/2022/12/MI_20221231_1672467986732.png)  

<details> <summary>Nerd Font</summary>

Nerd Fonts 是一个使用大量字体图标来解决程序员在开发过程中缺少合适字体的问题的项目。

[Nerd Font 项目下载页面](https://www.nerdfonts.com/font-downloads)

官方建议 Windows 系统下使用 TTF 可变字重字体。
</details>

## 安装 PowerShell（Core）

1. [安装 PowerShell](https://learn.microsoft.com/zh-cn/powershell/scripting/install/installing-powershell-on-windows?view=powershell-7.3)

    ```shell
    winget install --id Microsoft.Powershell --source winget
    ```

    重启终端即可在下拉选项中看到 PowerShell。

2. 编辑`settings.json`中的`list`属性，清爽面板

    ![图 5](https://cdn.jsdelivr.net/gh/Z-404/imageHost@main/2022/12/MI_20221231_1672471499280.png)  

<details> <summary>模块列表</summary>

posh-git
PSFzf
Terminal-Icons
z
</details>

<details> <summary>模块命令</summary>

1. [Find-Module](https://learn.microsoft.com/zh-cn/powershell/module/powershellget/find-module?view=powershell-7.3)

2. [Install-Module](https://learn.microsoft.com/zh-cn/powershell/module/powershellget/install-module?view=powershell-7.3)

3. [Import-Module](https://learn.microsoft.com/zh-cn/powershell/module/microsoft.powershell.core/import-module?view=powershell-7.3)
   
4. [Get-InstalledModule](https://learn.microsoft.com/zh-cn/powershell/module/powershellget/get-installedmodule?view=powershell-7.3)

5. [Uninstall-Module](https://learn.microsoft.com/zh-cn/powershell/module/powershellget/uninstall-module?view=powershell-7.3)

6. [Update-Module](https://learn.microsoft.com/zh-cn/powershell/module/powershellget/update-module?view=powershell-7.3)

例：
1. 安装并导入模块：

    ```shell
    Install-Module Terminal-Icons 
    # 将下行添加到 $PROFILE
    Import-Module Terminal-Icons
    ```

2. 使用命令管道卸载 Modules

    ```shell
    Get-InstalledModule <module-name> | Uninstall-Module
    ```
</details>

<details> <summary>模块位置</summary>

1. 查看默认模块位置，键入：

    ```shell
    $Env:PSModulePath
    ```

2. 添加默认模块位置,使用以下命令格式:

    ```shell
    $Env:PSModulePath = $Env:PSModulePath + ";<path>"
    ```

    例如，若要添加 `C:\ps-test\Modules` 目录，键入：

    ```shell
    $Env:PSModulePath + ";C:\ps-test\Modules"
    ```

3. 若要在 Linux 或 MacOS 上添加默认模块位置，请使用以下命令格式：

    ```shell
    $Env:PSModulePath += ":<path>"
    ```

    例如，将 `/usr/local/Fabrikam/Modules` 目录添加到 PSModulePath 环境变量的值，键入：

    ```shell
    $Env:PSModulePath += ":/usr/local/Fabrikam/Modules"
    ```
</details>

## 安装 "Oh My Posh"

[Oh My Posh](https://ohmyposh.dev/docs/) 是一个自定义提示符（Prompt）引擎，适用于任何能够使用函数或变量调整提示字符串的 shell。

1. 使用`winget`命令安装 oh-my-posh：

    ```shell
    winget install JanDeDobbeleer.OhMyPosh -s winget
    # 重新启动 shell 以重新加载 PATH
    ```

    注意，在 Windows 中 OhMyPosh 以`oh-my-posh.exe`的形式存在。
    
    不要使用`Install-Module`的方式安装 OhMyPosh。

2. 配置主题：[montys](https://ohmyposh.dev/docs/themes#montys)，在`$PROFILE`中添加下行：

    ```ps1
    oh-my-posh init pwsh --config 'C:\Users\31880\AppData\Local\Programs\oh-my-posh\themes\montys.omp.json' | Invoke-Expression
    ```

    ![图 7](https://cdn.jsdelivr.net/gh/Z-404/imageHost@main/2022/12/MI_20221231_1672474836003.png)  

    - `Get-PoshThemes`命令可以下载并展示所有发布主题，这不是必须的，你也可以使用远程URL配置主题。
    
    - 名称中带有 minimal 的主题不需要 Nerd 字体。

## 导入模块

### Terminal-Icons：添加文件类型图标

1. 安装 Terminal-IconsTerminal-Icons 模块：

    ```shell
    Install-Module -Name Terminal-Icons -Repository PSGallery
    ```

2. 在`$PROFILE`中添加 (edit with "code $PROFILE")：

    ```ps1
    Import-Module Terminal-Icons
    ```

    - `Show-TerminalIconsTheme`命令展示所有图标样式。

![图 6](https://cdn.jsdelivr.net/gh/Z-404/imageHost@main/2022/12/MI_20221231_1672473344173.png)  

### PSReadLine：增强命令行编辑

1. 安装 PSReadLine 模块：

    ```shell
    Install-Module -Name PSReadLine -AllowClobber -Force
    ```
    
    - PowerShell 7 自带 PSReadLine 模块，导入即可。

2. 在`$PROFILE`中添加 (edit with "code $PROFILE"):

    ```ps1
    Import-Module PSReadLine

    # Set PSReadLine
    Set-PSReadLineOption -PredictionSource History 
    Set-PSReadLineOption -PredictionViewStyle ListView 
    Set-PSReadLineOption -EditMode Windows 
    ```
![图 8](https://cdn.jsdelivr.net/gh/Z-404/imageHost@main/2022/12/MI_20221231_1672475197151.png)  

### post-git: 为提示符增加 Git 状态摘要

...你已经可以自己写了吧？

### z：短路径

...同上

## 其他优化

1. Alias

1. 安装 Scoop

    ```shell
    iwr -useb get.scoop.sh | iex 
    ```
1. 安装 neovim、gcc、sudo

    ```shell
    scoop install neovim gcc sudo curl jq
    ```

## 总结

Power shell 在 Windows 中和 cmd 不完全兼容，在 linux 中也。。。

有一丝诡异

连个虚拟机挺好用的，传文件就 scp（可选 WinSCP），编辑用 VSCode 远程连接插件。

## code $PROFILE

```ps1
using namespace System.Management.Automation
using namespace System.Management.Automation.Language
 
if ($host.Name -eq 'ConsoleHost')
{
    Import-Module PSReadLine
}
#Import-Module PSColors
Import-Module posh-git
Import-Module -Name Terminal-Icons
#Import-Module oh-my-posh
Import-Module z
set-alias desktop "Desktop.ps1"
set-alias ll ls
set-alias grep findstr
#Set-Theme ParadoxGlucose
#Set-PoshPrompt -theme "D:\Dropbox\poshv3.json"

#oh-my-posh --init --shell pwsh --config "C:\Users\31880\AppData\Local\Programs\oh-my-posh\themes\jandedobbeleer.omp.json" | Invoke-Expression
oh-my-posh --init --shell pwsh --config "C:\Users\31880\AppData\Local\Programs\oh-my-posh\themes\myMontys.omp.json" | Invoke-Expression


Register-ArgumentCompleter -Native -CommandName winget -ScriptBlock {
    param($wordToComplete, $commandAst, $cursorPosition)
        [Console]::InputEncoding = [Console]::OutputEncoding = $OutputEncoding = [System.Text.Utf8Encoding]::new()
        $Local:word = $wordToComplete.Replace('"', '""')
        $Local:ast = $commandAst.ToString().Replace('"', '""')
        winget complete --word="$Local:word" --commandline "$Local:ast" --position $cursorPosition | ForEach-Object {
            [System.Management.Automation.CompletionResult]::new($_, $_, 'ParameterValue', $_)
        }
}

# PowerShell parameter completion shim for the dotnet CLI
Register-ArgumentCompleter -Native -CommandName dotnet -ScriptBlock {
     param($commandName, $wordToComplete, $cursorPosition)
         dotnet complete --position $cursorPosition "$wordToComplete" | ForEach-Object {
            [System.Management.Automation.CompletionResult]::new($_, $_, 'ParameterValue', $_)
         }
 }

# ---


# This is an example profile for PSReadLine.
#
# This is roughly what I use so there is some emphasis on emacs bindings,
# but most of these bindings make sense in Windows mode as well.

# Searching for commands with up/down arrow is really handy.  The
# option "moves to end" is useful if you want the cursor at the end
# of the line while cycling through history like it does w/o searching,
# without that option, the cursor will remain at the position it was
# when you used up arrow, which can be useful if you forget the exact
# string you started the search on.
Set-PSReadLineKeyHandler -Key UpArrow -Function HistorySearchBackward
Set-PSReadLineKeyHandler -Key DownArrow -Function HistorySearchForward

# This key handler shows the entire or filtered history using Out-GridView. The
# typed text is used as the substring pattern for filtering. A selected command
# is inserted to the command line without invoking. Multiple command selection
# is supported, e.g. selected by Ctrl + Click.
Set-PSReadLineKeyHandler -Key F7 `
                         -BriefDescription History `
                         -LongDescription 'Show command history' `
                         -ScriptBlock {
    $pattern = $null
    [Microsoft.PowerShell.PSConsoleReadLine]::GetBufferState([ref]$pattern, [ref]$null)
    if ($pattern)
    {
        $pattern = [regex]::Escape($pattern)
    }

    $history = [System.Collections.ArrayList]@(
        $last = ''
        $lines = ''
        foreach ($line in [System.IO.File]::ReadLines((Get-PSReadLineOption).HistorySavePath))
        {
            if ($line.EndsWith('`'))
            {
                $line = $line.Substring(0, $line.Length - 1)
                $lines = if ($lines)
                {
                    "$lines`n$line"
                }
                else
                {
                    $line
                }
                continue
            }

            if ($lines)
            {
                $line = "$lines`n$line"
                $lines = ''
            }

            if (($line -cne $last) -and (!$pattern -or ($line -match $pattern)))
            {
                $last = $line
                $line
            }
        }
    )
    $history.Reverse()

    $command = $history | Out-GridView -Title History -PassThru
    if ($command)
    {
        [Microsoft.PowerShell.PSConsoleReadLine]::RevertLine()
        [Microsoft.PowerShell.PSConsoleReadLine]::Insert(($command -join "`n"))
    }
}


# CaptureScreen is good for blog posts or email showing a transaction
# of what you did when asking for help or demonstrating a technique.
Set-PSReadLineKeyHandler -Chord 'Ctrl+d,Ctrl+c' -Function CaptureScreen

# The built-in word movement uses character delimiters, but token based word
# movement is also very useful - these are the bindings you'd use if you
# prefer the token based movements bound to the normal emacs word movement
# key bindings.
Set-PSReadLineKeyHandler -Key Alt+d -Function ShellKillWord
Set-PSReadLineKeyHandler -Key Alt+Backspace -Function ShellBackwardKillWord
Set-PSReadLineKeyHandler -Key Alt+b -Function ShellBackwardWord
Set-PSReadLineKeyHandler -Key Alt+f -Function ShellForwardWord
Set-PSReadLineKeyHandler -Key Alt+B -Function SelectShellBackwardWord
Set-PSReadLineKeyHandler -Key Alt+F -Function SelectShellForwardWord

#region Smart Insert/Delete

# The next four key handlers are designed to make entering matched quotes
# parens, and braces a nicer experience.  I'd like to include functions
# in the module that do this, but this implementation still isn't as smart
# as ReSharper, so I'm just providing it as a sample.

Set-PSReadLineKeyHandler -Key '"',"'" `
                         -BriefDescription SmartInsertQuote `
                         -LongDescription "Insert paired quotes if not already on a quote" `
                         -ScriptBlock {
    param($key, $arg)

    $quote = $key.KeyChar

    $selectionStart = $null
    $selectionLength = $null
    [Microsoft.PowerShell.PSConsoleReadLine]::GetSelectionState([ref]$selectionStart, [ref]$selectionLength)

    $line = $null
    $cursor = $null
    [Microsoft.PowerShell.PSConsoleReadLine]::GetBufferState([ref]$line, [ref]$cursor)

    # If text is selected, just quote it without any smarts
    if ($selectionStart -ne -1)
    {
        [Microsoft.PowerShell.PSConsoleReadLine]::Replace($selectionStart, $selectionLength, $quote + $line.SubString($selectionStart, $selectionLength) + $quote)
        [Microsoft.PowerShell.PSConsoleReadLine]::SetCursorPosition($selectionStart + $selectionLength + 2)
        return
    }

    $ast = $null
    $tokens = $null
    $parseErrors = $null
    [Microsoft.PowerShell.PSConsoleReadLine]::GetBufferState([ref]$ast, [ref]$tokens, [ref]$parseErrors, [ref]$null)

    function FindToken
    {
        param($tokens, $cursor)

        foreach ($token in $tokens)
        {
            if ($cursor -lt $token.Extent.StartOffset) { continue }
            if ($cursor -lt $token.Extent.EndOffset) {
                $result = $token
                $token = $token -as [StringExpandableToken]
                if ($token) {
                    $nested = FindToken $token.NestedTokens $cursor
                    if ($nested) { $result = $nested }
                }

                return $result
            }
        }
        return $null
    }

    $token = FindToken $tokens $cursor

    # If we're on or inside a **quoted** string token (so not generic), we need to be smarter
    if ($token -is [StringToken] -and $token.Kind -ne [TokenKind]::Generic) {
        # If we're at the start of the string, assume we're inserting a new string
        if ($token.Extent.StartOffset -eq $cursor) {
            [Microsoft.PowerShell.PSConsoleReadLine]::Insert("$quote$quote ")
            [Microsoft.PowerShell.PSConsoleReadLine]::SetCursorPosition($cursor + 1)
            return
        }

        # If we're at the end of the string, move over the closing quote if present.
        if ($token.Extent.EndOffset -eq ($cursor + 1) -and $line[$cursor] -eq $quote) {
            [Microsoft.PowerShell.PSConsoleReadLine]::SetCursorPosition($cursor + 1)
            return
        }
    }

    if ($null -eq $token -or
        $token.Kind -eq [TokenKind]::RParen -or $token.Kind -eq [TokenKind]::RCurly -or $token.Kind -eq [TokenKind]::RBracket) {
        if ($line[0..$cursor].Where{$_ -eq $quote}.Count % 2 -eq 1) {
            # Odd number of quotes before the cursor, insert a single quote
            [Microsoft.PowerShell.PSConsoleReadLine]::Insert($quote)
        }
        else {
            # Insert matching quotes, move cursor to be in between the quotes
            [Microsoft.PowerShell.PSConsoleReadLine]::Insert("$quote$quote")
            [Microsoft.PowerShell.PSConsoleReadLine]::SetCursorPosition($cursor + 1)
        }
        return
    }

    # If cursor is at the start of a token, enclose it in quotes.
    if ($token.Extent.StartOffset -eq $cursor) {
        if ($token.Kind -eq [TokenKind]::Generic -or $token.Kind -eq [TokenKind]::Identifier -or 
            $token.Kind -eq [TokenKind]::Variable -or $token.TokenFlags.hasFlag([TokenFlags]::Keyword)) {
            $end = $token.Extent.EndOffset
            $len = $end - $cursor
            [Microsoft.PowerShell.PSConsoleReadLine]::Replace($cursor, $len, $quote + $line.SubString($cursor, $len) + $quote)
            [Microsoft.PowerShell.PSConsoleReadLine]::SetCursorPosition($end + 2)
            return
        }
    }

    # We failed to be smart, so just insert a single quote
    [Microsoft.PowerShell.PSConsoleReadLine]::Insert($quote)
}

Set-PSReadLineKeyHandler -Key '(','{','[' `
                         -BriefDescription InsertPairedBraces `
                         -LongDescription "Insert matching braces" `
                         -ScriptBlock {
    param($key, $arg)

    $closeChar = switch ($key.KeyChar)
    {
        <#case#> '(' { [char]')'; break }
        <#case#> '{' { [char]'}'; break }
        <#case#> '[' { [char]']'; break }
    }

    $selectionStart = $null
    $selectionLength = $null
    [Microsoft.PowerShell.PSConsoleReadLine]::GetSelectionState([ref]$selectionStart, [ref]$selectionLength)

    $line = $null
    $cursor = $null
    [Microsoft.PowerShell.PSConsoleReadLine]::GetBufferState([ref]$line, [ref]$cursor)
    
    if ($selectionStart -ne -1)
    {
      # Text is selected, wrap it in brackets
      [Microsoft.PowerShell.PSConsoleReadLine]::Replace($selectionStart, $selectionLength, $key.KeyChar + $line.SubString($selectionStart, $selectionLength) + $closeChar)
      [Microsoft.PowerShell.PSConsoleReadLine]::SetCursorPosition($selectionStart + $selectionLength + 2)
    } else {
      # No text is selected, insert a pair
      [Microsoft.PowerShell.PSConsoleReadLine]::Insert("$($key.KeyChar)$closeChar")
      [Microsoft.PowerShell.PSConsoleReadLine]::SetCursorPosition($cursor + 1)
    }
}

Set-PSReadLineKeyHandler -Key ')',']','}' `
                         -BriefDescription SmartCloseBraces `
                         -LongDescription "Insert closing brace or skip" `
                         -ScriptBlock {
    param($key, $arg)

    $line = $null
    $cursor = $null
    [Microsoft.PowerShell.PSConsoleReadLine]::GetBufferState([ref]$line, [ref]$cursor)

    if ($line[$cursor] -eq $key.KeyChar)
    {
        [Microsoft.PowerShell.PSConsoleReadLine]::SetCursorPosition($cursor + 1)
    }
    else
    {
        [Microsoft.PowerShell.PSConsoleReadLine]::Insert("$($key.KeyChar)")
    }
}

Set-PSReadLineKeyHandler -Key Backspace `
                         -BriefDescription SmartBackspace `
                         -LongDescription "Delete previous character or matching quotes/parens/braces" `
                         -ScriptBlock {
    param($key, $arg)

    $line = $null
    $cursor = $null
    [Microsoft.PowerShell.PSConsoleReadLine]::GetBufferState([ref]$line, [ref]$cursor)

    if ($cursor -gt 0)
    {
        $toMatch = $null
        if ($cursor -lt $line.Length)
        {
            switch ($line[$cursor])
            {
                <#case#> '"' { $toMatch = '"'; break }
                <#case#> "'" { $toMatch = "'"; break }
                <#case#> ')' { $toMatch = '('; break }
                <#case#> ']' { $toMatch = '['; break }
                <#case#> '}' { $toMatch = '{'; break }
            }
        }

        if ($toMatch -ne $null -and $line[$cursor-1] -eq $toMatch)
        {
            [Microsoft.PowerShell.PSConsoleReadLine]::Delete($cursor - 1, 2)
        }
        else
        {
            [Microsoft.PowerShell.PSConsoleReadLine]::BackwardDeleteChar($key, $arg)
        }
    }
}

#endregion Smart Insert/Delete

# Sometimes you enter a command but realize you forgot to do something else first.
# This binding will let you save that command in the history so you can recall it,
# but it doesn't actually execute.  It also clears the line with RevertLine so the
# undo stack is reset - though redo will still reconstruct the command line.
Set-PSReadLineKeyHandler -Key Alt+w `
                         -BriefDescription SaveInHistory `
                         -LongDescription "Save current line in history but do not execute" `
                         -ScriptBlock {
    param($key, $arg)

    $line = $null
    $cursor = $null
    [Microsoft.PowerShell.PSConsoleReadLine]::GetBufferState([ref]$line, [ref]$cursor)
    [Microsoft.PowerShell.PSConsoleReadLine]::AddToHistory($line)
    [Microsoft.PowerShell.PSConsoleReadLine]::RevertLine()
}

# Insert text from the clipboard as a here string
Set-PSReadLineKeyHandler -Key Ctrl+V `
                         -BriefDescription PasteAsHereString `
                         -LongDescription "Paste the clipboard text as a here string" `
                         -ScriptBlock {
    param($key, $arg)

    Add-Type -Assembly PresentationCore
    if ([System.Windows.Clipboard]::ContainsText())
    {
        # Get clipboard text - remove trailing spaces, convert \r\n to \n, and remove the final \n.
        $text = ([System.Windows.Clipboard]::GetText() -replace "\p{Zs}*`r?`n","`n").TrimEnd()
        [Microsoft.PowerShell.PSConsoleReadLine]::Insert("@'`n$text`n'@")
    }
    else
    {
        [Microsoft.PowerShell.PSConsoleReadLine]::Ding()
    }
}

# Sometimes you want to get a property of invoke a member on what you've entered so far
# but you need parens to do that.  This binding will help by putting parens around the current selection,
# or if nothing is selected, the whole line.
Set-PSReadLineKeyHandler -Key 'Alt+(' `
                         -BriefDescription ParenthesizeSelection `
                         -LongDescription "Put parenthesis around the selection or entire line and move the cursor to after the closing parenthesis" `
                         -ScriptBlock {
    param($key, $arg)

    $selectionStart = $null
    $selectionLength = $null
    [Microsoft.PowerShell.PSConsoleReadLine]::GetSelectionState([ref]$selectionStart, [ref]$selectionLength)

    $line = $null
    $cursor = $null
    [Microsoft.PowerShell.PSConsoleReadLine]::GetBufferState([ref]$line, [ref]$cursor)
    if ($selectionStart -ne -1)
    {
        [Microsoft.PowerShell.PSConsoleReadLine]::Replace($selectionStart, $selectionLength, '(' + $line.SubString($selectionStart, $selectionLength) + ')')
        [Microsoft.PowerShell.PSConsoleReadLine]::SetCursorPosition($selectionStart + $selectionLength + 2)
    }
    else
    {
        [Microsoft.PowerShell.PSConsoleReadLine]::Replace(0, $line.Length, '(' + $line + ')')
        [Microsoft.PowerShell.PSConsoleReadLine]::EndOfLine()
    }
}

# Each time you press Alt+', this key handler will change the token
# under or before the cursor.  It will cycle through single quotes, double quotes, or
# no quotes each time it is invoked.
Set-PSReadLineKeyHandler -Key "Alt+'" `
                         -BriefDescription ToggleQuoteArgument `
                         -LongDescription "Toggle quotes on the argument under the cursor" `
                         -ScriptBlock {
    param($key, $arg)

    $ast = $null
    $tokens = $null
    $errors = $null
    $cursor = $null
    [Microsoft.PowerShell.PSConsoleReadLine]::GetBufferState([ref]$ast, [ref]$tokens, [ref]$errors, [ref]$cursor)

    $tokenToChange = $null
    foreach ($token in $tokens)
    {
        $extent = $token.Extent
        if ($extent.StartOffset -le $cursor -and $extent.EndOffset -ge $cursor)
        {
            $tokenToChange = $token

            # If the cursor is at the end (it's really 1 past the end) of the previous token,
            # we only want to change the previous token if there is no token under the cursor
            if ($extent.EndOffset -eq $cursor -and $foreach.MoveNext())
            {
                $nextToken = $foreach.Current
                if ($nextToken.Extent.StartOffset -eq $cursor)
                {
                    $tokenToChange = $nextToken
                }
            }
            break
        }
    }

    if ($tokenToChange -ne $null)
    {
        $extent = $tokenToChange.Extent
        $tokenText = $extent.Text
        if ($tokenText[0] -eq '"' -and $tokenText[-1] -eq '"')
        {
            # Switch to no quotes
            $replacement = $tokenText.Substring(1, $tokenText.Length - 2)
        }
        elseif ($tokenText[0] -eq "'" -and $tokenText[-1] -eq "'")
        {
            # Switch to double quotes
            $replacement = '"' + $tokenText.Substring(1, $tokenText.Length - 2) + '"'
        }
        else
        {
            # Add single quotes
            $replacement = "'" + $tokenText + "'"
        }

        [Microsoft.PowerShell.PSConsoleReadLine]::Replace(
            $extent.StartOffset,
            $tokenText.Length,
            $replacement)
    }
}

# This example will replace any aliases on the command line with the resolved commands.
Set-PSReadLineKeyHandler -Key "Alt+%" `
                         -BriefDescription ExpandAliases `
                         -LongDescription "Replace all aliases with the full command" `
                         -ScriptBlock {
    param($key, $arg)

    $ast = $null
    $tokens = $null
    $errors = $null
    $cursor = $null
    [Microsoft.PowerShell.PSConsoleReadLine]::GetBufferState([ref]$ast, [ref]$tokens, [ref]$errors, [ref]$cursor)

    $startAdjustment = 0
    foreach ($token in $tokens)
    {
        if ($token.TokenFlags -band [TokenFlags]::CommandName)
        {
            $alias = $ExecutionContext.InvokeCommand.GetCommand($token.Extent.Text, 'Alias')
            if ($alias -ne $null)
            {
                $resolvedCommand = $alias.ResolvedCommandName
                if ($resolvedCommand -ne $null)
                {
                    $extent = $token.Extent
                    $length = $extent.EndOffset - $extent.StartOffset
                    [Microsoft.PowerShell.PSConsoleReadLine]::Replace(
                        $extent.StartOffset + $startAdjustment,
                        $length,
                        $resolvedCommand)

                    # Our copy of the tokens won't have been updated, so we need to
                    # adjust by the difference in length
                    $startAdjustment += ($resolvedCommand.Length - $length)
                }
            }
        }
    }
}

# F1 for help on the command line - naturally
Set-PSReadLineKeyHandler -Key F1 `
                         -BriefDescription CommandHelp `
                         -LongDescription "Open the help window for the current command" `
                         -ScriptBlock {
    param($key, $arg)

    $ast = $null
    $tokens = $null
    $errors = $null
    $cursor = $null
    [Microsoft.PowerShell.PSConsoleReadLine]::GetBufferState([ref]$ast, [ref]$tokens, [ref]$errors, [ref]$cursor)

    $commandAst = $ast.FindAll( {
        $node = $args[0]
        $node -is [CommandAst] -and
            $node.Extent.StartOffset -le $cursor -and
            $node.Extent.EndOffset -ge $cursor
        }, $true) | Select-Object -Last 1

    if ($commandAst -ne $null)
    {
        $commandName = $commandAst.GetCommandName()
        if ($commandName -ne $null)
        {
            $command = $ExecutionContext.InvokeCommand.GetCommand($commandName, 'All')
            if ($command -is [AliasInfo])
            {
                $commandName = $command.ResolvedCommandName
            }

            if ($commandName -ne $null)
            {
                Get-Help $commandName -ShowWindow
            }
        }
    }
}


#
# Ctrl+Shift+j then type a key to mark the current directory.
# Ctrj+j then the same key will change back to that directory without
# needing to type cd and won't change the command line.

#
$global:PSReadLineMarks = @{}

Set-PSReadLineKeyHandler -Key Ctrl+J `
                         -BriefDescription MarkDirectory `
                         -LongDescription "Mark the current directory" `
                         -ScriptBlock {
    param($key, $arg)

    $key = [Console]::ReadKey($true)
    $global:PSReadLineMarks[$key.KeyChar] = $pwd
}

Set-PSReadLineKeyHandler -Key Ctrl+j `
                         -BriefDescription JumpDirectory `
                         -LongDescription "Goto the marked directory" `
                         -ScriptBlock {
    param($key, $arg)

    $key = [Console]::ReadKey()
    $dir = $global:PSReadLineMarks[$key.KeyChar]
    if ($dir)
    {
        cd $dir
        [Microsoft.PowerShell.PSConsoleReadLine]::InvokePrompt()
    }
}

Set-PSReadLineKeyHandler -Key Alt+j `
                         -BriefDescription ShowDirectoryMarks `
                         -LongDescription "Show the currently marked directories" `
                         -ScriptBlock {
    param($key, $arg)

    $global:PSReadLineMarks.GetEnumerator() | % {
        [PSCustomObject]@{Key = $_.Key; Dir = $_.Value} } |
        Format-Table -AutoSize | Out-Host

    [Microsoft.PowerShell.PSConsoleReadLine]::InvokePrompt()
}

# Auto correct 'git cmt' to 'git commit'
Set-PSReadLineOption -CommandValidationHandler {
    param([CommandAst]$CommandAst)

    switch ($CommandAst.GetCommandName())
    {
        'git' {
            $gitCmd = $CommandAst.CommandElements[1].Extent
            switch ($gitCmd.Text)
            {
                'cmt' {
                    [Microsoft.PowerShell.PSConsoleReadLine]::Replace(
                        $gitCmd.StartOffset, $gitCmd.EndOffset - $gitCmd.StartOffset, 'commit')
                }
            }
        }
    }
}

# `ForwardChar` accepts the entire suggestion text when the cursor is at the end of the line.
# This custom binding makes `RightArrow` behave similarly - accepting the next word instead of the entire suggestion text.
Set-PSReadLineKeyHandler -Key RightArrow `
                         -BriefDescription ForwardCharAndAcceptNextSuggestionWord `
                         -LongDescription "Move cursor one character to the right in the current editing line and accept the next word in suggestion when it's at the end of current editing line" `
                         -ScriptBlock {
    param($key, $arg)

    $line = $null
    $cursor = $null
    [Microsoft.PowerShell.PSConsoleReadLine]::GetBufferState([ref]$line, [ref]$cursor)

    if ($cursor -lt $line.Length) {
        [Microsoft.PowerShell.PSConsoleReadLine]::ForwardChar($key, $arg)
    } else {
        [Microsoft.PowerShell.PSConsoleReadLine]::AcceptNextSuggestionWord($key, $arg)
    }
}

# Cycle through arguments on current line and select the text. This makes it easier to quickly change the argument if re-running a previously run command from the history
# or if using a psreadline predictor. You can also use a digit argument to specify which argument you want to select, i.e. Alt+1, Alt+a selects the first argument
# on the command line. 
Set-PSReadLineKeyHandler -Key Alt+a `
                         -BriefDescription SelectCommandArguments `
                         -LongDescription "Set current selection to next command argument in the command line. Use of digit argument selects argument by position" `
                         -ScriptBlock {
    param($key, $arg)
  
    $ast = $null
    $cursor = $null
    [Microsoft.PowerShell.PSConsoleReadLine]::GetBufferState([ref]$ast, [ref]$null, [ref]$null, [ref]$cursor)
  
    $asts = $ast.FindAll( {
        $args[0] -is [System.Management.Automation.Language.ExpressionAst] -and
        $args[0].Parent -is [System.Management.Automation.Language.CommandAst] -and
        $args[0].Extent.StartOffset -ne $args[0].Parent.Extent.StartOffset
      }, $true)
  
    if ($asts.Count -eq 0) {
        [Microsoft.PowerShell.PSConsoleReadLine]::Ding()
        return
    }
    
    $nextAst = $null

    if ($null -ne $arg) {
        $nextAst = $asts[$arg - 1]
    }
    else {
        foreach ($ast in $asts) {
            if ($ast.Extent.StartOffset -ge $cursor) {
                $nextAst = $ast
                break
            }
        } 
        
        if ($null -eq $nextAst) {
            $nextAst = $asts[0]
        }
    }

    $startOffsetAdjustment = 0
    $endOffsetAdjustment = 0

    if ($nextAst -is [System.Management.Automation.Language.StringConstantExpressionAst] -and
        $nextAst.StringConstantType -ne [System.Management.Automation.Language.StringConstantType]::BareWord) {
            $startOffsetAdjustment = 1
            $endOffsetAdjustment = 2
    }
  
    [Microsoft.PowerShell.PSConsoleReadLine]::SetCursorPosition($nextAst.Extent.StartOffset + $startOffsetAdjustment)
    [Microsoft.PowerShell.PSConsoleReadLine]::SetMark($null, $null)
    [Microsoft.PowerShell.PSConsoleReadLine]::SelectForwardChar($null, ($nextAst.Extent.EndOffset - $nextAst.Extent.StartOffset) - $endOffsetAdjustment)
}


Set-PSReadLineOption -PredictionSource History
Set-PSReadLineOption -PredictionViewStyle ListView
Set-PSReadLineOption -EditMode Windows


# This is an example of a macro that you might use to execute a command.
# This will add the command to history.
Set-PSReadLineKeyHandler -Key Ctrl+Shift+b `
                         -BriefDescription BuildCurrentDirectory `
                         -LongDescription "Build the current directory" `
                         -ScriptBlock {
    [Microsoft.PowerShell.PSConsoleReadLine]::RevertLine()
    [Microsoft.PowerShell.PSConsoleReadLine]::Insert("dotnet build")
    [Microsoft.PowerShell.PSConsoleReadLine]::AcceptLine()
}

Set-PSReadLineKeyHandler -Key Ctrl+Shift+t `
                         -BriefDescription BuildCurrentDirectory `
                         -LongDescription "Build the current directory" `
                         -ScriptBlock {
    [Microsoft.PowerShell.PSConsoleReadLine]::RevertLine()
    [Microsoft.PowerShell.PSConsoleReadLine]::Insert("dotnet test")
    [Microsoft.PowerShell.PSConsoleReadLine]::AcceptLine()
}
```

## Themes

1. myMontys

    ```json
    {
    "$schema": "https://raw.githubusercontent.com/JanDeDobbeleer/oh-my-posh/main/themes/schema.json",
    "blocks": [
        {
        "alignment": "left",
        "segments": [
            {
            "background": "#003543",
            "foreground": "#00c983",
            "leading_diamond": "\ue0b6",
            "style": "diamond",
            "template": "{{ .Icon }} {{ .HostName }} ",
            "type": "os"
            },
            {
            "background": "#DA627D",
            "foreground": "#ffffff",
            "powerline_symbol": "\ue0b0",
            "properties": {
                "folder_icon": "\uf115",
                "folder_separator_icon": "\\",
                "home_icon": "\uf7db",
                "style": "full"
            },
            "style": "powerline",
            "template": " <#000>\uf07b \uf553</> {{ .Path }} ",
            "type": "path"
            },
            {
            "background": "#FCA17D",
            "foreground": "#ffffff",
            "powerline_symbol": "\ue0b0",
            "properties": {
                "branch_icon": " <#ffffff>\ue0a0 </>",
                "fetch_stash_count": true,
                "fetch_status": false,
                "fetch_upstream_icon": true
            },
            "style": "powerline",
            "template": " \u279c ({{ .UpstreamIcon }}{{ .HEAD }}{{ if gt .StashCount 0 }} \uf692 {{ .StashCount }}{{ end }}) ",
            "type": "git"
            },
            {
            "background": "#76b367",
            "foreground": "#ffffff",
            "powerline_symbol": "\ue0b0",
            "style": "powerline",
            "template": " \ue718 {{ if .PackageManagerIcon }}{{ .PackageManagerIcon }} {{ end }}{{ .Full }} ",
            "type": "node"
            },
            {
            "background": "#83769c",
            "foreground": "#ffffff",
            "powerline_symbol": "\ue0b0",
            "properties": {
                "always_enabled": true
            },
            "style": "powerline",
            "template": " \ufbab {{ .FormattedMs }} ",
            "type": "executiontime"
            },
            {
            "background": "#2e9599",
            "background_templates": [
                "{{ if gt .Code 0 }}red{{ end }}"
            ],
            "foreground": "#ffffff",
            "powerline_symbol": "\ue0b0",
            "properties": {
                "always_enabled": true
            },
            "style": "diamond",
            "template": " {{ if gt .Code 0 }}\uf525{{ else }}\uf469{{ end }}",
            "trailing_diamond": "\ue0b4",
            "type": "exit"
            }
        ],
        "type": "prompt"
        },
        {
        "alignment": "right",
        "segments": [
            {
            "foreground": "#f36943",
            "foreground_templates": [
                "{{if eq \"Charging\" .State.String}}#40c4ff{{end}}",
                "{{if eq \"Discharging\" .State.String}}#ff5722{{end}}",
                "{{if eq \"Full\" .State.String}}#00c983{{end}}"
            ],
            "properties": {
                "charged_icon": "\ue22f ",
                "charging_icon": "\ue234 ",
                "discharging_icon": "\ue231 "
            },
            "style": "plain",
            "template": " {{ if not .Error }}{{ .Icon }}{{ .Percentage }}{{ end }}{{ .Error }}\uf295  ",
            "type": "battery"
            },
            {
            "foreground": "#26C6DA",
            "style": "plain",
            "properties": {
                "time_format": "Mon Jan 2 15:04:05 2006"
            },
            "template": " <b>{{ .CurrentDate | date .Format }}</b>",
            "type": "time"
            }
        ],
        "type": "prompt"
        },
        {
        "alignment": "left",
        "newline": true,
        "segments": [
            {
            "foreground": "#cd5e42",
            "style": "plain",
            "template": "\ue231 ",
            "type": "root"
            },
            {
            "foreground": "#CD4277",
            "style": "plain",
            "template": " <#45F1C2><b>\u26a1</b></><b>{{ .UserName }}</b> <#26C6DA>\u276f</><#45F1C2>\u276f</>",
            "type": "text"
            }
        ],
        "type": "prompt"
        }
    ],
    "final_space": true,
    "version": 2
    }
    ```