.. role:: red
.. role:: bred

Lab 8.1: Troubleshoot
---------------------

It is important to first define "normal" behavior. If the SSL Orchestrator
deployment process was successful, it will be possible to access remote
Internet sites from the client workstation without issue, and HTTPS sites
appear to have a locally-trusted, re-issued server certificate. This would be
considered normal behavior. If any of these do not happen, use the tools below
to troubleshoot.

Below is a reasonably-ordered list of troubleshooting steps.

- If the SSL Orchestrator deployment process fails, review the ensuing error
  message. It would be impossible to list here all of the possible error
  messages and their meanings, but often enough the messages will reveal the
  issue.

- Re-review the lab steps for any missing or misconfigured settings.

- Enable debug logging in the SSL Orchestrator configuration. Tail the APM log
  from a BIG-IP command line or from the logs page in the management UI. Debug
  logging will very often reveal important issues. Specifically, it will
  indicate traffic classification matches, mismatches or deployment issues.

  .. code-block:: bash

     tail –f /var/log/apm

     tail -f /var/log/restnoded/restnoded.log

     tail -f /vr/log/restjavad.0.log

- If the SSL Orchestrator deployment process succeeds, but traffic isn't
  flowing through the environment made evident by lack of access to remote
  sites from the client:

  - Ensure that the client is properly configured to either default route to
    the ingress VLAN and self-IP of the BIG-IP for transparent proxy access or
    has the correct browser proxy settings defined for explicit proxy access.

  - Ensure that traffic is flowing to the BIG-IP from the client with a
    tcpdump capture at the ingress interface.

  - Review the LTM configuration created by the SSL Orchestrator.
    Specifically, look at the pools and respective monitors for any failures.

  - Isolate service chain services. If at least one service chain has been
    created, and debug logging indicates that traffic is matching this chain,
    remove all but one service from that chain and test. Add one service back
    at a time until traffic flow stops. If a single added service breaks
    traffic flow, this service will typically be the culprit.

  - If a broken service is identified, insert probes to verify inbound and
    outbound traffic flow. Inline services will have a source (S) VLAN and
    destination (D) VLAN, and ICAP and receive only services will each have a
    single source VLAN. Insert a tcpdump capture at each VLAN in order to
    determine if traffic is getting to the device, and if traffic is leaving
    the device through its outbound interface.

  - If no service chains are defined, it may be necessary to remove all of the
    defined services and re-create them one-by-one to validate flow through
    the built-in All chain. If a broken service is identified, insert tcpdump
    probes as described above.

  - If traffic is flowing through all of the security devices, insert a
    tcpdump probe at the egress point to verify traffic is leaving the BIG-IP
    to the gateway router.

  - If traffic is flowing to the gateway router, perform a more extensive
    packet analysis to determine if SSL if failing between the BIG-IP egress
    point and the remote server.

    .. code-block:: bash

       tcpdump –i 0.0:nnn –nn –Xs0 –vv –w <file.pcap> <any additional filters>

    Then either export this capture to WireShark or send to ssldump:

    .. code-block:: bash

       ssldump –nr <file.pcap> -H –S crypto > text-file.txt

  - If the WireShark or ssldump analysis verifies an SSL issue:

    - Plug the site's address into the SSLLabs.com server test site at:

      https://www.ssllabs.com/ssltest/

      This report will indicate any specific SSL requirements that this site
      has.

    - Verify that the SSL Orchestrator server SSL profiles (two of them) have
      the correct cipher string to match the requirements of this site. To do
      that, perform the following command at the BIG-IP command line:

      .. code-block:: bash

         tmm --clientciphers 'CIPHER STRING AS DISPLAYED IN SERVER SSL PROFILES'

    - Further SSL/TLS issues are beyond the depth of this lab guide. Seek
      assistance.

- If all else fails, seek assistance.
