# IOI Print Filter

This is the source of the print filter which was used at IOI 2015. It's purpose
is to ensure that every print job is gets a cover page at the start, a final
page at the back, and that every page in between is watermarked with the ID of
the user who printed the job.

Installation is non-trivial, and you probably want to read the `ioi-filter`
script carefully before using it. The filter is designed to operate on a
central CUPS server managing print jobs for all contestant PCs.

Steps to install this include:
 * Copy the `ioi-filter` to /usr/lib/cups/filter/. Ensure it has execute
   permissions and is writable only by root (otherwise CUPS will complain).
 * Copy `aux-template.html` and `html2pdf.js` some directory, and edit
   `$SCRIPT_DIR` in `ioi-filter` to point to that directory.
 * Write a script `ioi-print-contestant-details` which searches in its
   command-line arguments for an IP address such as
   `job-originating-host-name=a.b.c.d` and outputs a bunch of environment
   variables which will be used in the template. Put it in `$SCRIPT_DIR`.
   Non-ASCII characters should be HTML escaped -- in python this can be done
   with `s.encode('ascii', 'xmlcharrefreplace')`. Sample output from this
   script would look like:
```sh
CONTESTANT_ID='AUS01'
CONTESTANT_NAME='Ch&#228rlie W. Budgie'
CONTESTANT_COUNTRY='Australia'
CONTESTANT_LOCATION='E17'
```
 * `touch /var/lib/ioiprint.log`, `chown lp:root` and `chmod 644` it.
 * Now is the hard part. You need to modify the PPD file(s) of your printers so
   that this script is called. It depends heavily on your printer and it's PPD
   file, but in essence you want to add the following lines somewhere:
```
*cupsFilter: "application/vnd.cups-postscript 0 ioi-filter"
*cupsFilter: "application/vnd.cups-pdf 0 ioi-filter"
```
 * The exact location may require some experiementation. Ideally it's after the
   prologue, but before any main business. Or if there are any other
   `cupsFilters` lines in the PPD, you probably want to put it after them, or
   maybe before. Do some experimentation.
 * This assumes that your printer natively accepts PS files. If it actually
   uses PDF instead, then you should edit the relevant lines at the end of
   `ioi-filter` to output PDF instead of Postscript.

On the contestant machines, you should install the remote printer with a
Generic Printer which generates PS or PDF files.
