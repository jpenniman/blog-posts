---
layout: post
title: Using PDB (Debug Symbols) When Compiling in Release Mode
date: 2011-03-07 09:45:00.000000000 -05:00
author: Jason M Penniman
excerpt: Bit by yet another not-so-well documented "feature" in Microsoft .Net. &nbsp;We
  were compiling in release mode for QA, since we've seen other behavioral differences
  between debug and release. &nbsp;We also wanted to emit the line numbers in the
  stack trace so the engineers would have a better idea of what caused the error.
  We added the...
category: Development
---
<p>Bit by yet another not-so-well documented "feature" in Microsoft .Net.  We were compiling in release mode for QA, since we've seen other behavioral differences between debug and release.  We also wanted to emit the line numbers in the stack trace so the engineers would have a better idea of what caused the error.  We added the /degub:pdbonly option, but left the /optimize+.  We noticed that some of the errors when compared to the line numbers didn't make sense.  It turned out that most of the stack trace was wrong... or at least wrong when compared to the original source code.<br /><br />If you want to emit debug symbols, ie. pdb files, when compiling in release mode, you have to disable optimizations.  Compiler optimizations will often reorganize instructions for the most efficient execution.  As a result, any stack trace will produce unexpected line numbers... you'll get the line number in the newly optimized code, not the line in your original source code.</p>
<div> </div>
<h3>Disabling Optimizations and Enabling pdb only</h3>
<div>If compiling at the command line, or with nant:</div>
<div> </div>
<pre><span>/debug:pdbonly /optimize-</span></pre>
<div>or simply</div>
<pre><span>/debug:pdbonly</span></pre>
<div> </div>
<div>In Visual Studio:</div>
<div> </div>
<div>Under project properties:</div>
<div><ol>
<li>Uncheck "Enable Optimization"</li>
<li>In the "Advanced ..." settings, choose "pdb-only" under Output-&gt;Debug Info.</li>
</ol></div>
<div> </div>
<div><a href="https://lh3.googleusercontent.com/-uCCXpfhlRAk/TXUJ_sCbK3I/AAAAAAAAAPA/YPIzS-Ua6yQ/s1600/pdb_release.jpg"><img src="https://lh3.googleusercontent.com/-uCCXpfhlRAk/TXUJ_sCbK3I/AAAAAAAAAPA/YPIzS-Ua6yQ/s1600/pdb_release.jpg" border="0" /></a></div>
<div> </div>
<div> </div>
<div> </div>
<p>The statement in the Microsoft docs: "It is possible to combine the /optimize and <a href="http://msdn.microsoft.com/en-us/library/8cw0bt21(v=VS.90).aspx">/debug</a> options." -- is not entirely accurate.<br /><br /></p>
<div>Based on Microsoft's documentation, I'm placing this in the "Just because you can, doesn't mean you should." category. <br />
<div> </div>
<div>References:</div>
<div><a href="http://stackoverflow.com/questions/935726/getting-line-number-from-pdb-in-release-mode">http://stackoverflow.com/questions/935726/getting-line-number-from-pdb-in-release-mode</a></div>
<div>
<div><a href="http://msdn.microsoft.com/en-us/library/t0hfscdc(v=VS.90).aspx">http://msdn.microsoft.com/en-us/library/t0hfscdc(v=VS.90).aspx</a></div>
<div> </div>
<div><br />
<div> </div>
</div>
</div>
</div>
<div><img src="https://blogger.googleusercontent.com/tracker/7838145634981365715-4373818276281315530?l=jpenniman.blogspot.com" border="0" width="1" height="1" /></div>
