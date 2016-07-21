use Rex -feature => ['1.0'];
use Getopt::Long;
use String::Util qw( nocontent );
use Term::ReadPassword;

my $runtime_zip;
my @devices;
my $reboot;
my $uname;

GetOptions(
  "runtime-zip=s"   => \$runtime_zip,
  "device=s"        => \@devices,
  'sudo-password'   => \&sudo_password_handler,
  'uname=s'         => \$uname,
) or die "Error in command line arguments\n";

if (nocontent $uname) {
  user "hb";
}
else {
  user $uname;
}

#password "test";

group devices => qw(

), @devices;

task "reboot", group => "devices", sub {
  #restart device if requested
  sudo "shutdown -r now";
};

task "deploy", group => "devices", sub {

  my $hb_dir = "HappyBrackets";

  run "mkdir -p $hb_dir";

  # Upload deps archive
  file "HappyBracketsRuntime.zip",
    source => "$runtime_zip";

  #clean runtime dir
  sudo "killall java";
  run "rm -rf $hb_dir";

  #unpack
  run "unzip -d $hb_dir HappyBracketsRuntime.zip";
  run "rm HappyBracketsRuntime.zip";

  #run "(java -cp libs/HB.jar net.happybrackets.device.DeviceMain &) &",
  #  cwd => $hb_dir;

  # run "echo Run was here >> rex-log";
  # sudo "echo sudo was here >> rex-log";
  sudo "./run.sh",
    cwd => "$hb_dir/scripts";

};

task "setup", group => "devices", sub {

  run "echo Started task setup " . localtime . " >> rex-log";

  sudo TRUE;

  #zeroconf
  pkg ["avahi-daemon", "avahi-discover", "libnss-mdns"],
    ensure => "present";

  #unzip
  pkg "unzip",
    ensure => "present";

  #Java

  #get the key?
  run "apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys EEA14886";

  repository "add" => "webupd8team-java",
    url         => "http://ppa.launchpad.net/webupd8team/java/ubuntu",
    sign_key    => "keyserver.ubuntu.com",
    distro      => "xenial",
    repository  => "main",
    source      => 1;

  #accept Oracle Liscence
  run "echo oracle-java8-installer shared/accepted-oracle-license-v1-1 select true | /usr/bin/debconf-set-selections";

  pkg "oracle-java8-installer",
    ensure => "present";

  #TODO: handle the setup of wifi credentials

  #setup rc.local
  file "/etc/rc.local",

    owner     => 'root',
    group     => 'root',
    mode      => '755',
    content => template("files/rc.local.tpl", env => {
        user    => $uname,
    });

  sudo FALSE;

  run "echo Finished task setup " . localtime . " >> rex-log";

};

sub sudo_password_handler {
  sudo_password password_arg_handler(@_);
}

sub password_arg_handler {
  my ($opt_name, $opt_value) = @_;

  if (nocontent $opt_value or $opt_value = 1) {
    return read_password("$opt_name: ");
  }
  else {
    return $opt_value;
  }
}
