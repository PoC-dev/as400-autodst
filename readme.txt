These are two ILE CL Scripts for older OS/400 releases without the ability to
self-adjust to daylight saving time.

It is free software; you can redistribute it and/or modify it under the terms
 of the GNU General Public License as published by the Free Software
 Foundation; either version 2 of the License, or (at your option) any later
 version.

It is distributed in the hope that it will be useful, but WITHOUT ANY
 WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR
 A PARTICULAR PURPOSE. See the GNU General Public License for more details.

You should have received a copy of the GNU General Public License along with
 it; if not, write to the Free Software Foundation, Inc., 59 Temple Place,
 Suite 330, Boston, MA 02111-1307 USA or get it at
 http://www.gnu.org/licenses/gpl.html


What is it?
===========
OS/400 V4 - and earlier - can't automatically adjust to DST. Two times a year,
the operator is required to do this adjustment by hand. Later releases finally
learned to do that without intervention. I've written two not very elegant ILE
CL scripts to take over this duty, and relieve the operator from remembering
to adjust the clock.

They do a crude check with the QUTCOFFSET system variable to decide if
adjustment is needed, or not. A date/time check is *not* done for determining
the need for adjustment! This at least prevents a script from running multiple
times and adjusting the clock hour by hour each run.

All variables as are hard coded for CET. Change as needed.


How to use?
===========
First: Upload.

Preferred use is a command line FTP client. Log onto your machine (as QSECOFR),
switch to ASCII mode and upload the files.

 put enddst.clle QGPL/QCLSRC.ENDDST
 put strdst.clle QGPL/QCLSRC.STRDST

End the FTP session.

Sign on as QSECOFR to a 5250 session and copy/paste the two commands to
set metadata correctly, and compile the source into objects.

 CHGPFM FILE(QCLSRC) MBR(ENDDST) SRCTYPE(CLLE) TEXT('End Daylight Saving Time')
 CHGPFM FILE(QCLSRC) MBR(STRDST) SRCTYPE(CLLE) TEXT('Start Daylight Saving Time')
 CRTBNDCL PGM(QGPL/ENDDST) SRCFILE(QCLSRC)
 CRTBNDCL PGM(QGPL/STRDST) SRCFILE(QCLSRC)

Assuming the clock and QUTCOFFSET is set correctly for now, set up scheduled
jobs for the next run of both scripts:

 ADDJOBSCDE JOB(ENDDST) CMD(CALL PGM(ENDDST)) FRQ(*ONCE) SCDDATE('31.10.21') SCDTIME('3:00') MSGQ(QSYSOPR) TEXT('Switch Clock to CET.')
 ADDJOBSCDE JOB(STRDST) CMD(CALL PGM(STRDST)) FRQ(*ONCE) SCDDATE('27.03.22') SCDTIME('2:00') MSGQ(QSYSOPR) TEXT('Switch Clock to CEST.')

From now on, the scripts will create new one-time scheduled jobs after
adjusting the clock and QUTCOFFSET variables. Even if the machine is shut down
and not running when switchover should occur, the default value *SBMRLS will
run the job at the next IPL.

If you are lazy, wait until adjustment should take place and simply run the
appropriate script. Adjustment will be done and the job for next year created.

End of March 2022, this solution proved to work.


ToDos
=====
Insert error handling. Currently I don't know what can go wrong, though. :-)
Move all adjustable values to variables at the top of the script.
Unify both scripts into one, while a command line parameter decides str or end.


vim: textwidth=78 autoindent
