These are two CL Scripts for older OS/400 releases without the ability to self-adjust to daylight saving time.

## License.
This document is part of as400-autodst, to be found on [GitHub](https://github.com/PoC-dev/as400-autodst). Its content is subject to the [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/) license, also known as *Attribution-ShareAlike 4.0 International*. The project itself is subject to the GNU Public License version 2,or (at your option) any later version.

## What is it?
OS/400 V4 - and earlier - can't automatically adjust to DST. Two times a year, the operator is required to do this adjustment by hand. Later releases finally learned to do that without intervention. I've written two not very elegant ILE CL scripts to take over this duty, and relieve the operator from remembering to adjust the clock.

They do a crude check with the `qutcoffset` system variable to decide if adjustment is needed, or not. A date/time check is *not* done for determining the need for adjustment! This at least prevents a script from running multiple times and adjusting the clock hour by hour each run. Contrary, manual ensurance is required to make it run only when DST starts or ends.

All variables as are hard coded for CET. Change as needed.

Running the jobs creates scheduled job entries for the next year's run.

## How to use?
First: Upload.

Preferred use is a command line FTP client. Log onto your machine (as *QSECOFR*), switch to ASCII mode and upload the files to the by default existing source PF.
```
put enddst.clp qgpl/qclsrc.enddst
put strdst.clp qgpl/qclsrc.strdst
quote rcmd chgpfm file(qgpl/qclsrc) mbr(enddst) srctype(clle) text('End Daylight Saving Time')
quote rcmd chgpfm file(qgpl/qclsrc) mbr(strdst) srctype(clle) text('Start Daylight Saving Time')
quit
```
Sign on as *qsecofr* to a 5250 session and copy/paste the two commands to compile the source into objects.
```
crtbndcl pgm(qgpl/enddst)
crtbndcl pgm(qgpl/strdst)
```
Assuming the clock and `qutcoffset` is set correctly for now, set up scheduled jobs for the next run of both scripts. Example:
```
addjobscde job(enddst) cmd(call pgm(enddst)) frq(*once) scddate('31.10.21') scdtime('3:00') msgq(qsysopr) text('Switch Clock to CET.')
addjobscde job(strdst) cmd(call pgm(strdst)) frq(*once) scddate('27.03.22') scdtime('2:00') msgq(qsysopr) text('Switch Clock to CEST.')
```
**Note:** There's no point in creating jobs scheduled for running in the pastime. Adjust dates accordingly to the current year.

From now on, the scripts will create new one-time scheduled jobs after adjusting the clock and `qutcoffset` variables. Even if the machine is shut down and not running when switchover should occur, the default value `*sbmrls` for the retry action parameter `rcyacn` will ensure the job is run at the next IPL.

If you are lazy, wait until adjustment should take place and simply run the appropriate script. Adjustment will be done and the job for next year created.

**Note:** You need to do the same for the opposite switchover!

Since end of March 2022, this solution proved to work flawlessly.

## ToDos.
Laziness prohibited those being done so far: Works good enough…
- Insert error handling. Currently I don't know what can go wrong, though. :-)
- Move all adjustable values to variables at the top of the scripts.
- Merge both scripts into one, while a command line parameter decides str or end.

----

2024-09-05 poc@pocnet.net
