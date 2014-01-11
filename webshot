#!/usr/bin/perl
$version = "0.3";

use LWP;
use Path::Class qw/file/;
use Imager::Screenshot 'screenshot';
use X11::GUITest qw/
	WaitWindowViewable
	SendKeys/;
use Getopt::Std;
use POSIX qw/strftime/;
use Socket;


if (@ARGV == 0 )
{
 print "_} Webshotter version $version {_\n";
 print "[-] Command Line Options\n";
 print "[-] Required Command Line Options\n";
 print "\t-i infile\n";
 #print "\t -o outfile\n";
 print "\t-t Title in Output\n";
 print "[-] Optional Command Line Options\n";
 print "\t-u Define User-Agent\n";
 exit 0;
}


  my %args;	
  # Specify Title for Output
  getopt('itu', \%args);

  $infile = $args{i};
  $title = $args{t};
  $uagent = $args{u};

#Declare Default user controlled vars
 $CWD = $ENV{'PWD'};
 $sleep = 1;
 $directory = $CWD ."/data";
 $lwtimeout = "20";
 $errornum = 0;


# loop through directory_NUM to create 
# next numerical directory
$dirnum = 0;
$havedir = 0;
while ( $havedir == 0 )
{
  $newdir = $directory . "_" . $dirnum;
  if ( -e $newdir )
  {
    #print "$newdir exists\n";
    $dirnum++;
  }
  else 
  {
     #print "$newdir does NOT exist\n";
     $directory = $newdir;
     $wsnum = $dirnum;
     $havedir = 1;
  }
	
}

	
 unless(-e $directory or mkdir $directory) {
	die "Unable to create $directory\n";
 }


open(FILE, $infile) || die("Could not open $infile");
#print "opened $infile\n";


$date = strftime('%m-%d-%Y %H:%M:%S %Z',localtime);

# Create HTML Header Content
$outfile = "webshot_" . $dirnum . ".htm";
open(OUTFILE, '>>' . $outfile);
print OUTFILE "<html><body>  <style>
   table td {border:solid 1px #fab; word-wrap:break-word;}
   </style>\n";
print OUTFILE "<h2>$title <br />Ran on: $date</h2>";
print OUTFILE "<table border=1 bgcolor='#E1E1E1'>";
print OUTFILE "<tr><td><h3>Internal Host List:</h3><br>\n";


#
# Create links to individual target
# sites in html file
#
while (<FILE>) {
 chomp ($_);
 if ( $_ =~ /https:/ )
 {
	$pname = $_;
  	$pname =~ s/https:\/\///;
	$pname =~ s/\//\./g;

 }
 else
 {
	$pname = $_;
	$pname =~ s/http:\/\///g;
	$pname =~ s/\//\./g;

 }
   

 print OUTFILE "<a href=#" . $pname . ">$_</a></br>\n ";
}


#
# Create links to external sites
# Open in new tab
#
print OUTFILE "</td><td><h3>External Site Links: </h3><br>";

seek FILE, 0, 0;
while (<FILE>) {
 chomp ($_);
   
 print OUTFILE "<a href=" . $_ . " target='_blank'>$_</a></br>\n ";
}

print OUTFILE "</td></tr></table>";
print OUTFILE "</br></br>";

# return to beginning of file
seek FILE, 0, 0; 

while( <FILE> ) {
chomp($_);
print "[+] Grabbing Site: $_ \n";



 our $ua = LWP::UserAgent->new( );

 $ua->ssl_opts( SSL_verify_mode => IO::Socket::SSL::SSL_VERIFY_NONE, 
 SSL_hostname => '', verify_hostname => 0 );
 $ua->timeout($lwtimeout);
 
 if( $uagent)
 {
   $ua->agent($uagent);
   print "\t[+] User-Agent: " . $ua->agent . "\n";
 }


 my $resp = $ua->get( $_ );


if( ! $resp->is_error )
{
	print "\t[+] Following HTTP Redirects\n";
	$headerdata = '';
	$pname = $_;
 if ( $_ =~ /https:/ )
 {
	$pname = $_;
	$pname =~ s/https:\/\///g;
	$pname =~ s/\//\./g;
	$host = $pname;
	$host =~ s/\.$//g;


 }

 if ( $_ =~ /http:/ )
 {
	$pname = $_;
	$pname =~ s/http:\/\///g;
	$pname =~ s/\//\./g;
	$host = $pname;
	$host =~ s/\.$//g;

 }

print "Before host is $host\n";
#Resolve address DNS Records
#Very basic regex to determine if this is an IP address
if($host =~ /\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}/)
 {
     #print "$host is an ip address\n";
     $nhost = inet_aton($host);
     $ptr_record  = gethostbyaddr( $nhost, AF_INET);
     if ( ! $ptr_record )
     {
	$ptr_record = "No Record";
     }
     
    else
    {
     $ip_addy = gethostbyname($ptr_record);
     $host_record = inet_ntoa($ip_addy);
    }


 }
else #pname is a dns name, resolve
{
    $ip_addy = gethostbyname($host);
    $host_record = inet_ntoa($ip_addy);

     $nhost = inet_aton($ip_addy);
     $ptr_record  = gethostbyaddr( $nhost, AF_INET);
     if ( ! $ptr_record )
     {
	$ptr_record = "No Record";
     }
}
   
	
  	$imagename = $pname . ".png";
	$imagefile = $directory . "/" . $imagename;

#print "DEBUG: " . $pname . ", imagename " . $imagename . ", imagefile " . $imagefile . "\n";


	print OUTFILE "\n<table border=1 bgcolor='#E1E1E1'";
	print OUTFILE "style='table-layout: fixed; width: 100%;'>";
	print OUTFILE "<colgroup><col style='width: 50%;'></colgroup>";
	print OUTFILE "<tr><td><a name=\"$_\">$_</a>";
	print OUTFILE " <a target='_blank' href=$_>External Link</a></td></tr>";
	print OUTFILE "<tr><td><b><u>DNS Info</u>:</b><br>\n";
	print OUTFILE "<b>IP Address:</b> $host_record<br>\n";
	print OUTFILE "<b>PTR Record:</b> $ptr_record<br>\n";
	print OUTFILE "<b><u>Header</u>:</b><br>\n";


	open my $hh, '>', \$headerdata or die "Can't open variable: $!";

 #Build Header Info

	print $hh "<b>Final URI</b>:" . $resp->request()->uri() . "\n";
	#Location:
	print $hh "<b>Response Status Line</b>: " . $resp->status_line . "\n";
	print $hh "<b>Server</b>: " . $resp->header( "Server" ) . "\n";
	print $hh "<b>Redirects</b>: " . $resp->redirects() . "\n\n";
 if ( $_ =~ /https:/ )
 {
	print $hh "<b><u>SSL Cert Info</u></b><br>";
	print $hh "<b>Cert-Subject</b>: " . $resp->header("Client-SSL-Cert-Subject"). "\n";
	print $hh "<b>Cert-Issuer</b>: " . $resp->header("Client-SSL-Cert-Issuer"). "\n";
	print $hh "\n\n";
 }	
	print $hh "<b>Set-Cookie</b>: " . $resp->header("Set-Cookie"). "\n";
	print $hh "<b>Date</b>: " . $resp->header("Date"). "\n";	
	print $hh "<b>Title</b>: " . $resp->header("Title"). "\n";
	print $hh "<b>X-Powered-By</b>: " . $resp->header("X-Powered-By"). "\n";
	#print $hh "Name: " . $resp->header("");
	#print $hh "\nHeaders: " . $resp->headers()->as_string ;

	close $hh;
	#$headerdata =~ s/^/<b>/mg;
	#s/:/:<\/b>/g for $headerdata;
	s/\n/<br>\n/g for $headerdata;
	

	print OUTFILE $headerdata;
	print OUTFILE "</td><td><a href=". $imagefile .">";
	print OUTFILE "<img height=512 width=512 src=";
	print OUTFILE $imagefile . ">";
	print OUTFILE "</a></td></tr></table><br>\n\n\n\n\n";

	system("chromium " . $_ . " > /dev/null 2>&1 &");
	 print "\t[+] Sleeping for $sleep seconds for Browser to load\n";
	sleep($sleep);

 if ( $_ =~ /https:/ )
 {
  # Wait for window to be chromium viewable
  my ($windowid) = WaitWindowViewable('Chromium');
  if (!$windowid) {
    die ("Couldn't get window\n");
  }
  SendKeys('{TAB}');
  SendKeys('{ENT}');
 }


 $loadsleep = 5;
 print "\t[+] Sleeping for $loadsleep seconds for Page to load\n";
 sleep($loadsleep);


  my $img = screenshot();
  $img->write(file => $imagefile, type => 'png' ) || print "Failed: ",  $img->{ERRSTR}, "n";
  print "\t[+] Saved image to $imagename\n";

  print "\n";
}
 # END OF if error 
 
 else
 {
   print "\t[!] Skipping host $_ \n";
   print "\t[!] " . $resp->status_line . "\n";
   $failedSites[$errornum] = ($_  );
   $failedSites[$errornum][$errornum] = ($resp->status_line );
   $errornum ++;
 } 
}


	
	print OUTFILE "\n<br><br><br><br><br><table border=1 bgcolor='#E1E1E1'>";
	print OUTFILE "<tr><td><b><u><a name='failures'>Failed Site</a></u>:</b></td>";
	print OUTFILE "<td><b><u>Error Response</u>:</b><br></td></tr>\n";

for ( $i = 0; $i != $errornum; $i++ )
{
 $fsite = $failedSites[$i];
 $ferror = $failedSites[$i][$i];
 print OUTFILE "<tr><td><a href=\"$fsite\" target='_blank'>$fsite  </a></td>";
 print OUTFILE " <td>$ferror</td></tr>";
# print "[-] Error for " . $failedSites[$i] . " was " . $failedSites[$i][$i] . "\n";
}
print OUTFILE "</table>\n\n";


print "[+] Webshot v$version Finished\n";
print OUTFILE "</br></br></body></html>";

close(FILE);
close(OUTFILE);

system ("chromium $outfile > /dev/null 2>&1 &");



