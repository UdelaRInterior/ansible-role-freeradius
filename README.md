Freeradius
==========

Install and configure on debian 9:
  1- Freeradius with MySQL 
  2- Freeradius with LDAP

Requirements
------------

Case 1 
	MySQL local or remote [documentation](https://wiki.freeradius.org/guide/SQL-HOWTO-for-freeradius-3.x-on-Debian-Ubuntu)
Case 2
	Server Ldap local or remote [documentation](https://wiki.debian.org/LDAP/OpenLDAPSetup)

Role Variables
--------------

# defaults file for ansible-role-freeradius

freeradius_version: '3.0'
freeradius_clients_default_password:                     ## Password used to connect routers 

## multiple clients can be defined
freeradius_clients:                                      
  - name: localhost                                       
    ipaddr: 127.0.0.1                                    
    secret: '{{ freeradius_clients_default_password }}'   
    shortname: localhost                                 
    nastype: other                                       
    require_message_authenticator: no                    
    limit:
      max_connections: 16                                
      lifetime: 0                                        
      idle_timeout: 30                                   

## define domains that are redirected locally
freeradius_realm:
  - domain1
  - domain2
  - domain3

## define multiple proxies
freeradius_proxy:
  - name: localhost
    type: auth
    ipaddr: 127.0.0.1
    port: 1812
    secret: '{{ freeradius_clients_default_password }}'
    response_window: 20
    zombie_period: 40
    revive_interval: 120
    status_check: status-server
    check_interval: 30
    check_timeout: 4
    num_answers_to_alive: 3
    max_outstanding: 65536
    coa:
      irt: 2
      mrt: 16
      mrc: 5
      mrd: 30
    limit:
      max_connections: 16
      max_requests: 0
      lifetime: 0
      idle_timeout: 0

## define the user system
freeradius_system_users: mysql            ##ldap or mysql

## define the connection to MySQL
freeradius_mysql:
  database: freeradius
  host: mysqlserver
  user: master
  password: master

## define the connection to Ldap
freeradius_ldap:
  server: 'ldap.com'
  port: 389
  identity: 'cn=login,cn=admin,ou=group,dc=example,dc=net'
  password: ''
  base_dn: 'dc=example,dc=net'
  control_password: True
  group_membership_attribute: 'memberOf'
  group_filter: '(objectClass=posixGroup)'
  client_filter: '(objectClass=radiusClient)'
  client_attribute.ipaddr: 'radiusClientIdentifier'
  client_attribute.secret: 'radiusClientSecret'
  filter: !unsafe '(cn=%{%{Stripped-User-Name}:-%{User-Name}})'
  options_chase_referrals: yes
  options_rebind: yes
  options_res_timeout: 10
  options_srv_timelimit: 3
  options_net_timeout: 1
  options_idle: 60
  options_probes: 3
  options_interval: 3
  options_ldap_debug: '0x0028'
  pool_start: '${thread[pool].start_servers}'
  pool_min: '${thread[pool].min_spare_servers}'
  pool_max: '${thread[pool].max_servers}'
  pool_spare: '${thread[pool].max_spare_servers}'
  pool_uses: 0
  pool_retry_delay: 30
  pool_lifetime: 0
  pool_idle_timeout: 60

## define data for certificates
freeradius_password_ca: '{{ freeradius_vault_password_ca }}'
freeradius_code_country: UY
freeradius_state: Montevideo
freeradius_locality: Montevideo
freeradius_organization: Example Inc.
freeradius_email: admin@example.net
freeradius_common_name: server.edu.net

Example Playbook
----------------

    - hosts: servers
      roles:
         - uspdev.freeradius

Some checks:
------------

MySQL check:
    insert into radcheck (username,attribute,op,value) values("maria", "Cleartext-Password", ":=", "m123");
    radtest maria m123 localhost 0 mysecret
    tail -f /var/log/freeradius/radius.log
