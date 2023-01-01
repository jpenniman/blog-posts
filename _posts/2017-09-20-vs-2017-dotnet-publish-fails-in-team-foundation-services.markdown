---
layout: post
title: VS 2017 dotnet publish fails in Team Foundation Services
date: 2017-09-20 23:47:37.000000000 -04:00
author: Jason M Penniman
excerpt: <p>We had an interesting problem after upgrading an ASP.Net Core project from project.json to VS 2017 csproj. The build broke. A little background...</p>
category: Blog
---
<p>We had an interesting problem after upgrading an ASP.Net Core project from project.json to VS 2017 csproj.  The build broke.  A little background...</p>
<p>The solution contains a portable library project.  The build fails with the error...</p>
<p>error MSB4019: The imported project `"C:\Program Files\dotnet\sdk\2.0.0\Microsoft\Portable\v4.5\Microsoft.Portable.CSharp.targets"` was not found.</p>
<p>As it turns out, the new dotnet publish does not support framework libraries, only core and netstandard.  </p>
<p><a href="https://stackoverflow.com/questions/44195599/dotnet-publish-failing-using-net-core-1-1-and-new-vs-2017-project-format">https://stackoverflow.com/questions/44195599/dotnet-publish-failing-using-net-core-1-1-and-new-vs-2017-project-format</a></p>
<p>This wouldn't be so bad, if publish wasn't trying to do a build as well.  The new tooling doesn't have the --no-build option prior to 2.1 being released.</p>
<p><a href="https://github.com/dotnet/cli/issues/5331">https://github.com/dotnet/cli/issues/5331</a></p>
<p> </p>
<p>To correct it, we followed the SO post above.  We removed the dotnet publish step and added the publish switch to the msbuild arguments for the VisualStudio build step.</p>
<p>Vioala! working CICD pipeline again!</p>
<p> </p>
<p> </p>
