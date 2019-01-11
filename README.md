Ansible Role: postgresql-nsr-backup
===================================

This role is used for EMC Networker backup configuration for PostgreSQL databases.

Supported OS:
-------------
* RedHat
* CentOS
* OracleLinux

Requirements
------------

None

Role Variables
--------------

Available variables are listed below, along with default values (see `defaults/main.yml`):

    # configure NetWorker directive files (.nsr)
    postgresql_nsr_backup_configure_dot_nsr_files: true

    # datavg mountpoint prefix
    postgresql_nsr_backup_datavg_mountpoint_prefix: '{{ postgresql_datavg_mountpoint_prefix }}'  # /postgres/data

    # binvg mountpoint prefix
    postgresql_nsr_backup_binvg_mountpoint_prefix: '{{ postgresql_binvg_mountpoint_prefix }}'  # /postgres/apps

    # location of nsr scripts (backup, restore, etc.)
    postgresql_nsr_backup_nsr_scripts_directory: '{{ postgresql_pghomes_directory }}/nsr'
    postgresql_nsr_backup_nsr_scripts_log_directory: '{{ postgresql_nsr_backup_nsr_scripts_directory }}/log'

    # nsr scripts
    postgresql_nsr_backup_nsr_scripts_templates:
      - nsr_pg_backup.sh
      - nsr_pg_backup_wal_archives_emergency.sh
      - nsr_pg_backup_list.sh
      - nsr_pg_backup_probe.sh
      - nsr_pg_restore_instance.sh
      - nsr_pg_restore_wal_archives.sh
      - nsr_pg_restore_tablespace_map.sh

    # whether or not copy nsr_* scripts to /bin directory
    postgresql_nsr_backup_copy_root_scripts: true

    # backup defaults
    postgresql_nsr_backup_default_backup_retention:      # default (empty): derived from NetWorker savegroup definition
    #postgresql_nsr_backup_default_backup_retention: '4 weeks'
    postgresql_nsr_backup_default_backup_role: postgres
    postgresql_nsr_backup_default_backup_dir_prefix: /postgres/backup/nsrbackup
    postgresql_nsr_backup_default_archive_dir_prefix: '{{ postgresql_nsr_backup_datavg_mountpoint_prefix }}/archive'
    postgresql_nsr_backup_default_savegroup_prefix: 'foo_00_am1_pgsdd'
    postgresql_nsr_backup_default_arch_usage_perc_threshold: 60

    # whether or not copy nsr_pg_backup_wal_archives_emergency.sh script
    postgresql_nsr_backup_cron_copy_scripts: true

    # backup cron config
    postgresql_nsr_backup_cron_config:
      - name: "trigger the archived WAL files backup when archive filesystem utilisation is exceeded"
        job: "{{ postgresql_nsr_backup_nsr_scripts_directory }}/nsr_pg_backup_wal_archives_emergency.sh --threshold={{ postgresql_nsr_backup_default_arch_usage_perc_threshold }} >/dev/null 2>&1"
        minute: 5,20,35,50
        state: present
        disabled: no

    # whether or not manage cron jobs
    postgresql_nsr_backup_manage_cron_jobs: true

    # whether or not disable cron jobs (for example: maintenance/patching)
    postgresql_nsr_backup_cron_disable_jobs: false


Dependencies
------------

This role uses `postgresql` role.

Example Playbook
----------------

    - name: Configure Networker backups for PostgreSQL databases
      hosts: pg-servers
      become: true
      become_user: '{{ postgresql_user }}'
      roles:
        - { role: postgresql-nsr-backup, tags: postgresql_nsr_backup }

Inside `vars/main.yml` or `group_vars/..` or `host_vars/..`:

    #-------------------------------------------------
    # overrides role 'postgresql-nsr-backup' variables
    #-------------------------------------------------

    # location of nsr scripts (backup, restore, etc.)
    postgresql_nsr_backup_nsr_scripts_directory: '{{ postgresql_pghomes_directory }}/nsr'
    postgresql_nsr_backup_nsr_scripts_log_directory: '{{ postgresql_nsr_backup_nsr_scripts_directory }}/log'

    # backup defaults
    postgresql_nsr_backup_default_backup_retention: '4 weeks'
    postgresql_nsr_backup_default_backup_role: postgres
    postgresql_nsr_backup_default_backup_dir_prefix: /postgres/backup/nsrbackup
    postgresql_nsr_backup_default_savegroup_prefix: 'foo_00_am1_pgsdd'
    postgresql_nsr_backup_default_arch_usage_perc_threshold: 60

    # ... etc ...


License
-------

GPLv3 - GNU General Public License v3.0

Author Information
------------------

This role was created in 2018 by [Krzysztof Lewandowski](mailto:Krzysztof.Lewandowski@fastmail.fm).


