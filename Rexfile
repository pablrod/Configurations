
use strict;
use warnings;
use utf8;
use Rex::Commands::SCM;
use Const::Fast;
use Path::Tiny;

const my $user => 'Nabu';

set
  repository => 'PersonalDotFiles',
  url        => 'git://github.com/pablrod/dotfiles.git',
  type       => 'git';

desc 'Add RPM Fusion repositories (Free and NonFree)';
task 'AddRPMFusionRepositories', sub {

    my $fedora_version;
    run 'rpm -E %fedora', sub {
        ($fedora_version) = @_;
    };

    pkg 'dnf', ensure => 'latest';

    run "dnf repolist", sub {
        my ($stdout) = @_;
        if ( scalar( grep { $_ =~ "rpmfusion-free" || $_ =~ "rpmfusion-nonfree" } split /\n/, $stdout ) == 0 ) {
            install package => [
                    "http://download1.rpmfusion.org/free/fedora/rpmfusion-free-release-$fedora_version.noarch.rpm",
                    "http://download1.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$fedora_version.noarch.rpm"
            ];
        } else {
            Rex::Logger::info("RPMFusion already installed");
        }
      }

};

desc 'Install fedora packages';
task 'InstallPackages', sub {
    update_package_db;

    my @packages = qw(
      dnf
      binutils bsdtar glibc rpm-build bash bash-completion wget curl lftp autofs lynx nmap autojump fasd fish zsh autojump-zsh autojump-fish mosh rsync ack lsof strace
      subversion git valgrind git-svn cmake
      boost-devel boost-static postgresql-devel libstdc++-devel expat libpng libcurl-devel libasan libasan-static liblsan liblsan-static libtsan libtsan-static libubsan libubsan-static eigen3-devel root fftw-devel gsl-devel zeromq-devel cppzmq-devel kernel-devel
      VirtualBox akmod-VirtualBox libvirt docker docker-compose
      postgresql pgcenter
      openbox obmenu obconf
      xclip xterm xdg-utils
      gstreamer1-libav gstreamer1-vaapi gstreamer1-plugins-good gstreamer1-plugins-good-extras gstreamer1-plugins-ugly lame
      firefox vlc gvim gimp meld keepassx shutter gnuplot vim-perl-support
      eclipse eclipse-jdt eclipse-subclipse eclipse-cdt-sdk eclipse-cdt-tests eclipse-epic eclipse-valgrind eclipse-mylyn-builds eclipse-mylyn-builds-hudson eclipse-mylyn-tasks-web eclipse-mylyn-tasks-bugzilla eclipse-mylyn-tasks-trac eclipse-tm-terminal
      wxGTK3-devel webkitgtk3-devel graphviz graphviz-devel
      pgadmin3 wireshark wireshark-gnome pacmanager vagrant vagrant-libvirt virt-manager
    );

    pkg \@packages, ensure => 'latest';

};

desc 'Configure VIM';
task 'ConfigureVim', sub {
    file "/home/$user/.vimrc",
      source => '/tmp/dotfiles/vim/.vimrc',
      owner  => $user,
      group  => $user,
      mode   => 0644;

    file "/home/$user/.vim",
      ensure => 'directory',
      owner  => $user,
      group  => $user,
      mode   => 0755;

    file "/home/$user/.vim/filetype.vim",
      source => '/tmp/dotfiles/vim/.vim/filetype.vim',
      owner  => $user,
      group  => $user,
      mode   => 0644;
};

desc 'Configure some extra bash completions';
task 'ConfigureBashCompletions', sub {
    my ( $path_completion, $path_helpers );

    run 'pkg-config --variable=completionsdir bash-completion', sub {
        ($path_completion) = @_;
    };

    run 'pkg-config --variable=helpersdir bash-completion', sub {
        ($path_helpers) = @_;
    };

    file $path_completion . '/rex',
      source => '/tmp/dotfiles/bash/rex_completion',
      owner  => 'root',
      group  => 'root',
      mode   => 644;
};

desc 'Configure Bash Shell';
task 'ConfigureBash', sub {

    my $home = Path::Tiny::path("/home/$user");
    file $home->child('.bashrc.d'),
      ensure => 'directory',
      owner  => $user,
      group  => $user,
      mode   => 755;

    my @bashrc_files = Path::Tiny::path('/tmp/dotfiles/bash/bashrc.d')->children;
    for my $bashrc_file (@bashrc_files) {
        file $home->child('.bashrc.d')->child($bashrc_file->basename)->canonpath,
          source => $bashrc_file->canonpath,
          owner  => $user,
          group  => $user,
          mode   => 644;
    }

    file $home->child('.bashrc'),
      source => '/tmp/dotfiles/bash/bashrc',
      owner  => $user,
      group  => $user,
      mode   => 644;

    ConfigureBashCompletions();
};

desc 'Configure OpenBox';
task 'ConfigureOpenBox', sub {
    file "/home/$user/.config/openbox",
      ensure => "directory",
      owner  => $user,
      group  => $user,
      mode   => 755;

    file "/home/$user/.config/openbox/autostart",
      source => '/tmp/dotfiles/openbox/autostart',
      owner  => $user,
      group  => $user,
      mode   => 755;

    file "/home/$user/.config/openbox/menu.xml",
      source => '/tmp/dotfiles/openbox/menu.xml',
      owner  => $user,
      group  => $user,
      mode   => 0644;

    file "/home/$user/.config/openbox/rc.xml",
      source => '/tmp/dotfiles/openbox/rc.xml',
      owner  => $user,
      group  => $user,
      mode   => 0644;

};

desc 'Configure installed packages';
task 'ConfigurePackages', sub {
    checkout 'PersonalDotFiles', path => '/tmp/dotfiles';
    ConfigureVim();
    ConfigureBash();
    ConfigureOpenBox();
};

desc 'Configure workstation';
task 'ConfigureWorkstation', sub {
    AddRPMFusionRepositories();
    InstallPackages();
    ConfigurePackages();
};
