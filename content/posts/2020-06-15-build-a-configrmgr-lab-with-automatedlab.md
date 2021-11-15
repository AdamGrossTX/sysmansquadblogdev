---
title: Build a ConfigrMgr lab with AutomatedLab
author: Adam Cook
type: post
date: 2020-06-15T06:00:00+00:00
url: /2020/06/15/build-a-configrmgr-lab-with-automatedlab/
uag_style_timestamp-js:
  - 1592283825
categories:
  - Endpoint Management
  - MECM/MEMCM/SCCM
  - Powershell
  - Scripting
tags:
  - automatedlab

---
In this post I'll show you how to start building a ConfigMgr lab, for either Current Branch or Technical Preview, using AutomatedLab with Hyper-V. This approach is intended to be completely automated and "hands off" by calling a single script.

It downloads all the necessary files for you, including the CB or TP installation media. All you have to do is provide a Windows Server 2016/2019 ISO, but this can also be an evaluation copy.

This post was originally posted on the <a rel="noreferrer noopener" href="https://winadmins.chat" target="_blank">WinAdmin's</a> blog, however we've joined forces with SysManSquad and moved the content. Since that post, I've updated the code to support the 2002 baseline, several bug fixes and now support installing Technical Preview! I also intend to provide clearer instructions.

### What is AutomatedLab?

[AutomatedLab][1] is a PowerShell module for building lab environments with simple PowerShell code! The module can be found on [GitHub][2] and more documentation also on their [website][3].

The benefit of using AutomatedLab is the simplicity it offers to build and throw away environments of varying complexity. Also, it offers a huge range of functions to customise your lab to how you want it. 

Generally you create your build script, set it to run, come back some time later and your lab is complete exactly how you want it. That's exactly what I've done with this ConfigMgr CustomRole: run the script, leave it alone and come back to a fully functioning and independent environment.

If you're curious and want an introduction to AutomatedLab, [check out this post I wrote][4]. However it's not strictly necessary to follow along right now with this post if you have no prior understanding.

### Yet another ConfigMgr lab solution

In my quest for finding an automated way to build out a ConfigMgr lab, I found several excellent options:

  * [PSNewCMENV][5] - @onpremcloudguy
  * [CMHyperHydrate][6] - @AdamGrossTX
  * [CMBuild][7] - @skatterbrainzz
  * [ViaMonstra Hydration Kit][8] - @jarwidmark

I was mainly interested in something that was hands off and had opportunities for me to contribute to. With that I felt AutomatedLab was exactly what I wanted.

At the time I found AutomatedLab, the ConfigMgr / SCCM / MEMCM / MECM (ARGH!) CustomRole in AutomatedLab's repository only had version 1802, so I thought I'd chip in.

### Getting started

First things first, make sure you have got the latest version of the AutomatedLab module installed. Compare your installed version with what's available on the <a href="https://github.com/AutomatedLab/AutomatedLab/releases" target="_blank" rel="noreferrer noopener">AutomatedLab repository assets page</a>:

<div class="wp-block-codemirror-blocks-code-block code-block">
  <pre class="CodeMirror" data-setting="{&quot;mode&quot;:&quot;powershell&quot;,&quot;mime&quot;:&quot;application/x-powershell&quot;,&quot;theme&quot;:&quot;default&quot;,&quot;lineNumbers&quot;:true,&quot;styleActiveLine&quot;:true,&quot;lineWrapping&quot;:true,&quot;readOnly&quot;:true,&quot;showPanel&quot;:false,&quot;languageLabel&quot;:&quot;no&quot;,&quot;language&quot;:&quot;PowerShell&quot;,&quot;modeName&quot;:&quot;powershell&quot;}">PS C:\&gt; Get-Module "AutomatedLab" -ListAvailable</pre>
</div>

<div class="wp-block-image">
  <figure class="aligncenter size-medium"><a href="https://sysmansquad.com/wp-content/uploads/2020/06/ALInstalledModule.jpg" target="_blank" rel="noopener noreferrer"><img loading="lazy" width="300" height="76" src="https://sysmansquad.com/wp-content/uploads/2020/06/ALInstalledModule-300x76.jpg" alt="" class="wp-image-1364" srcset="https:/wp-content/uploads/2020/06/ALInstalledModule-300x76.jpg 300w, https:/wp-content/uploads/2020/06/ALInstalledModule-768x195.jpg 768w, https:/wp-content/uploads/2020/06/ALInstalledModule-100x25.jpg 100w, https:/wp-content/uploads/2020/06/ALInstalledModule-855x217.jpg 855w, https:/wp-content/uploads/2020/06/ALInstalledModule.jpg 905w" sizes="(max-width: 300px) 100vw, 300px" /></a><figcaption>Get installed version of AutomatedLab</figcaption></figure>
</div>

To get started you'll need two things:

  * The `CM-2002.ps1` script.
  * The `CM-2002` CustomRole folder and all of its contents, stored in your local AutomatedLab's CustomRole folder.

You can source these things from either my <a rel="noreferrer noopener" href="https://github.com/codaamok/posh" target="_blank">PoSH GitHub repository</a>, the <a href="https://github.com/AutomatedLab/AutomatedLab" target="_blank" rel="noreferrer noopener">AutomatedLab GitHub repository</a> or they're are bundled within the AutomatedLab installer so you probably already have them on disk right now.

However, there's a high probability that what's in my repository is going to contain the latest fixes than anywhere else. So let's roll with that.

Download using either `git` or just download ZIP from the repository's web page:

<div class="wp-block-codemirror-blocks-code-block code-block">
  <pre class="CodeMirror" data-setting="{&quot;mode&quot;:&quot;shell&quot;,&quot;mime&quot;:&quot;text/x-sh&quot;,&quot;theme&quot;:&quot;default&quot;,&quot;lineNumbers&quot;:true,&quot;styleActiveLine&quot;:true,&quot;lineWrapping&quot;:true,&quot;readOnly&quot;:true,&quot;showPanel&quot;:false,&quot;languageLabel&quot;:&quot;no&quot;,&quot;language&quot;:&quot;Shell&quot;,&quot;modeName&quot;:&quot;shell&quot;}">cd C:\path\to\where\you\want\to\download
git clone https://github.com/codaamok/PoSH.git</pre>
</div>

<div class="wp-block-image">
  <figure class="aligncenter size-medium"><img loading="lazy" width="300" height="271" src="https://sysmansquad.com/wp-content/uploads/2020/06/CM2002ALDownloadZip-300x271.jpg" alt="" class="wp-image-1354" srcset="https:/wp-content/uploads/2020/06/CM2002ALDownloadZip-300x271.jpg 300w, https:/wp-content/uploads/2020/06/CM2002ALDownloadZip-1024x925.jpg 1024w, https:/wp-content/uploads/2020/06/CM2002ALDownloadZip-768x694.jpg 768w, https:/wp-content/uploads/2020/06/CM2002ALDownloadZip-100x90.jpg 100w, https:/wp-content/uploads/2020/06/CM2002ALDownloadZip-855x772.jpg 855w, https:/wp-content/uploads/2020/06/CM2002ALDownloadZip-1234x1115.jpg 1234w, https:/wp-content/uploads/2020/06/CM2002ALDownloadZip.jpg 1303w" sizes="(max-width: 300px) 100vw, 300px" /><figcaption>Download as ZIP from <a href="https://github.com/codaamok/posh">https://github.com/codaamok/posh</a></figcaption></figure>
</div>

Now the content is downloaded, we must copy only the `CustomRole\CM-2002` folder to the correct location on disk so AutomatedLab can use the scripts inside it. The location of `CM-2002.ps1` isn't important. 

If you were to use PowerShell, the command would look like this:

<div class="wp-block-codemirror-blocks-code-block code-block">
  <pre class="CodeMirror" data-setting="{&quot;mode&quot;:&quot;powershell&quot;,&quot;mime&quot;:&quot;application/x-powershell&quot;,&quot;theme&quot;:&quot;default&quot;,&quot;lineNumbers&quot;:true,&quot;styleActiveLine&quot;:true,&quot;lineWrapping&quot;:true,&quot;readOnly&quot;:true,&quot;showPanel&quot;:false,&quot;languageLabel&quot;:&quot;no&quot;,&quot;language&quot;:&quot;PowerShell&quot;,&quot;modeName&quot;:&quot;powershell&quot;}">PS C:\&gt; Copy-Item -Path "C:\path\to\where\you\downloaded\PoSH\AutomatedLab\CustomRoles\CM-2002" -Destination "C:\LabSources\CustomRoles" -Recurse -Force</pre>
</div>

Make sure you have a Windows Server 2016 or 2019 ISO (Evaluation will be fine) in your `C:\LabSources\ISOs` directory.

Now you can just call `CM-2002.ps1` as is, without any parameters! It will download all the prerequisites and necessary binaries, and install everything for you.

Of course parameters are available to further customise your lab, however as is will give you a functional lab with the below configurations:

  * 1x AutomatedLab lab
      * Name: "CMLab01"
      * VMPath: _<drive>_:\AutomatedLab-VMs where _<drive>_ is the fastest drive available
      * AddressSpace: An unused and available subnet increasing 192.168.1.0 by 1 until one is found.
      * ExternalVMSwitch: Allows physical network access via Hyper-V external switch named "Internet". "Default Switch" is an acceptable value to leverage an independent virtual network behind NAT.
  * 1x Active Directory domain:
      * Domain: "sysmansquad.lab"
      * Username: "Administrator"
      * Password: "Somepass1"
  * 2x Hyper-V virtual machines:
      * Operating System: Windows Server 2019 (Desktop Experience)
      * 1x Domain Controller
          * Name: "DC01"
          * vCPU: 2
          * Max memory: 2GB
          * Roles: "RootDC", "Routing"
      * 1x Configuration&nbsp;Manager&nbsp;primary&nbsp;site&nbsp;server:
          * Name: "CM01"
          * vCPU: 4
          * Max memory: 8GB
          * Roles: "SQLServer2017"
          * CustomRoles: "CM-2002"
          * SiteCode: "P01"
          * SiteName: "CMLab01"
          * Version: "Latest"
          * LogViewer: "OneTrace"
          * Site system roles: MP, DP, SUP (inc WSUS), RSP, EP

### Customisations

The majority of the above are customisable via the variety of parameters available for `CM-2002.ps1`. 

Review the parameters available for `CM-2002.ps1` via `Get-Help C:\path\to\CM-2002.ps1` with parameters like `-Detailed`, `-Parameter <paramaternamehere>` etc. Also in my repository is a `CM-2002.md` markdown file which lays out all the parameters and examples.

A few examples of things you can customise using parameters of `CM-2002.ps1` are:

  * Which branch to use, whether Current Branch or Technical Preview
  * Domain
  * Use Windows Server 2016 or 2019, Standard or Datacenter
  * Username / password 
  * Enable/disable auto logon
  * Address space / subnet for the lab
  * Site code and name
  * Hyper-V setting such as vCPU count or memory settings per VM
  * Target Configuration Manager version
      * At the time of writing this, only 2002 is available. However when updates become available, you'll be able to specify 2002, 2006, 2010 or just simply "Latest"
      * For Technical Preview, you'll only ever be able to specify 2002 or "Latest".

If you wanted to apply scripted customisations to your lab, after the build is complete, then you can consider using `C:\LabSources\CustomRoles\CM-2002\CustomiseCM.ps1`.

### Adding servers or clients to the lab

Doing this is as simple as spinning up VMs as you normally would, making them reachable to other VMs in the lab and join them to the domain.

If you're thinking, "wait, what? I've automated this much but now I have to manually install more clients/servers?!"... then check out the **Adding or removing lab machines** section <a href="https://sysmansquad.com/2020/06/15/getting-started-with-automatedlab/" target="_blank" rel="noreferrer noopener">from this post</a>.

### Support

If you experience any issues, please do open an issue on my <a href="https://github.com/codaamok/posh" target="_blank" rel="noreferrer noopener">GitHub repository</a>! Alternatively you can ping me on Twitter (<a href="https://twitter.com/codaamok" target="_blank" rel="noreferrer noopener">@codaamok</a>).

 [1]: https://github.com/automatedlab/automatedlab
 [2]: https://github.com/AutomatedLab/AutomatedLab
 [3]: https://automatedlab.org
 [4]: https://sysmansquad.com/2020/06/15/getting-started-with-automatedlab/
 [5]: https://github.com/onpremcloudguy/PSNewCMENV
 [6]: https://github.com/AdamGrossTX/CMHyperHydrate
 [7]: https://github.com/Skatterbrainz/CMBuild
 [8]: https://deploymentresearch.com/hydration-kit-for-windows-server-2016-and-configmgr-current-technical-preview-branch/