Markdown Monster has support for Long Path names, but it requires some Windows configuration to enable this functionality.

Windows by default does not support path names over 260 characters (`MAX_PATH`) using the standard Windows file access APIs. This means that if you try to open or save files that have paths (folder + file name) that exceed 260 characters those files can fail to open in Markdown Monster.

> As of Markdown Monster 3.8.4 we automatically enable Windows Long Path support using the full Installer version, so Long Path support should always work. 
>
> If you're using the Portable version, run `mmcli --enable-windows-longpaths` to enable Windows Long Path support explicitly.

## Override Windows Settings for Long Path Support
Markdown Monster has support for long file names, **but it only works if the Windows operating system has been enabled for Long Paths** which is done via a registry or group policy setting.

This setting essentially enables Long Path support for applications that request it, using standard Windows APIs without changes. Unfortunately this is not a default setting in Windows, so it has to be manually enabled and each application has to explicitly support it. Markdown Monster does, although there are some limitations.

Let's look at how to enable Long Path support in Windows.

### Using mmCli
The easiest way to enable long paths on Windows is to the use the Markdown Monster `mmCli` command. This command is globally registered via your path, so you should be able to run it from anywhere.

You can use the following commands from the Markdown Monster CLI to toggle the Long Path support in Windows

```ps
# Turn on long paths
mmcli enable-windows-longpaths

# Turn them off
mmcli disable-windows-longpaths
```

### It Works - mostly
Once you've set one of these options restart Markdown Monster and you should now be able to open and save files that exceed `MAX_PATH` character paths. This applies to all file operations that work through Markdown Monster's internal UI to open files into the editor and save files like images, PDF or HTML files etc.

## Caveats: Not everything works!
Although this setting allows you to open and save documents and related assets using long paths, **other applications and related support tools and components may not work correctly with long file names**. Even Windows components like Windows Explorer and Powershell have issues with long file name support, so be weary. In Markdown Monster this may mean shelling out to folders or files with long paths may not work as the external applications might not work with the long paths.

There may also be some preview scenarios where long paths will not render in the Preview browser due to long path names for relative images or other embedded links. 

In short, Markdown Monster's internal features related to file operations work fine with Long Path support. But you may still run into potential failures with long paths when shelling out into other applications.