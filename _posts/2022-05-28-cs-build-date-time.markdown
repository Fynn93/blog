---
layout: post
title:  How to get Build Date and Time in C#
date:   2022-05-28 09:00:00 +0200
author: Fynn
---

You wonder how to get the Build Date and Time in C#?

This is how!

First add this to your csproj file:
```xml
<PropertyGroup>
  <SourceRevisionId>build$([System.DateTime]::Now.ToString("yyyyMMddHHmmss"))</SourceRevisionId>
</PropertyGroup>
```

Next, create file AssemblyHaxx.cs with the contents
```cs
using System;
using System.Collections.Generic;
using System.Globalization;
using System.Linq;
using System.Reflection;
using System.Runtime.InteropServices;
using System.Text;
using System.Threading.Tasks;

namespace MyNamespace
{
    public class AssemblyHaxx
    {
        public static DateTime GetBuildInfo(Assembly assembly)
        {
            const string BuildVersionMetadataPrefix = "+build";

            var attribute = assembly.GetCustomAttribute<AssemblyInformationalVersionAttribute>();
            if (attribute?.InformationalVersion != null)
            {
                var value = attribute.InformationalVersion;
                var index = value.IndexOf(BuildVersionMetadataPrefix);
                if (index > 0)
                {
                    value = value[(index + BuildVersionMetadataPrefix.Length)..];
                    DateTime.TryParseExact(value, "yyyyMMddHHmmss", CultureInfo.InvariantCulture, DateTimeStyles.AssumeLocal, out DateTime dateTime);
                    if (DateTime.TryParseExact(value, "yyyyMMddHHmmss", CultureInfo.InvariantCulture, DateTimeStyles.AssumeLocal, out var result))
                    {
                        return result;
                    }
                }
            }

            return default;
        }
    }
}
```
You will need to replace the namespace `MyNamespace` with your Namespace.

To get the String, you need to add this code to any `.cs` file.
```cs
DateTime BuildDate = AssemblyHaxx.GetBuildInfo(Assembly.GetExecutingAssembly());

string VersionString = $"WTU {Assembly.GetExecutingAssembly().GetName().Version} built on {string.Format("{0:MMMM}", BuildDate)} {string.Format("{0:dd}", BuildDate)} {BuildDate.Year} {string.Format("{0:HH}", BuildDate)}:{string.Format("{0:mm}", BuildDate)}:{string.Format("{0:ss}", BuildDate)}";
```

Finally, the String `VersionString` will be something like:
```
WTU 1.0.0.0 built on May 26 2022 21:27:32
```

Thank you for reading this Blog!