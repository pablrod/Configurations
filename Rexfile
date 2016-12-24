
use Rex::Commands::SCM;
use Const::Fast;

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
                    "http://download1.rpmfusion.org/free/fedora/rpmfusion-free-release-$version_fedora.noarch.rpm",
                    "http://download1.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$version_fedora.noarch.rpm"
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
      firefox gvim
      eclipse
      xclip xterm
      bash bash-completion wget curl
      boost-devel boost-static postgresql-devel
      postgresql
    );
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

desc 'Configure installed packages';
task 'ConfigurePackages', sub {
    checkout 'PersonalDotFiles', path => '/tmp/dotfiles';
    ConfigureVim();
    ConfigureBashCompletions();
};

desc 'Configure workstation';
task 'ConfigureWorkstation', sub {
    AddRPMFusionRepositories();
    InstallPackages();
    ConfigurePackages();
};
