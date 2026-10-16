Per-game uplink latency
======================

RMNET can send the selected Android application's IPv4/IPv6 UDP packets
without waiting for the uplink aggregation timer. It uses the existing
aggregation flush path, sending any buffered packets first. Modem flow
control and channel selection remain unchanged. No modem low-latency
channel is forced, and no packet marks or DSCP values are changed.

Control
-------

``/sys/module/rmnet_core/parameters/game_low_latency_uid`` accepts:

* An application UID (at least 10000): enable or renew a 15-second lease.
* ``off UID``: release only if that UID still owns the lease.
* ``-1``: unconditionally disable, for boot initialization or administration.

Reading returns the active UID, or -1 when disabled or expired. The default
is disabled. XiaomiParts renews the lease every five seconds while an
opted-in game is focused and the screen is unlocked, and releases it when
that condition ends or cellular data ceases to be the active network.
Conditional release prevents a departing user's service from clearing
another user's request. A service crash leaves at most 15 seconds of lease.

``game_low_latency_flushes`` in the same directory is a cumulative count
of matching packets sent through the flush path, not milliseconds saved.

Only full UDP sockets owned by the selected UID match. TCP, GSO packets,
packets without a full socket, and other UIDs retain their existing path.
A VPN normally owns the outer socket, so XiaomiParts does not request a
cellular lease for VPN networks. Shared-UID packages share socket ownership.

Wi-Fi uses the Android WIFI_MODE_FULL_LOW_LATENCY lock, attributed to the
foreground game's UID through WorkSource; no RMNET setting applies to Wi-Fi.

Validation on a device
----------------------

* Verify that opting in an active game sets its UID on cellular data.
* Send IPv4 and IPv6 UDP from that UID and check the packet counter.
  Compare another UID and the disabled case; neither should increment it.
* Check leaving the game, locking the screen, disabling profiles and
  switching between Wi-Fi and cellular data. Check expiry after a service
  crash, and reactivation when the selected game regains focus.
* Compare in-game RTT, jitter and packet loss with the same server, signal
  conditions and load. ICMP ping alone does not exercise this UDP path.

The change only removes local aggregation waiting where aggregation was
active. It cannot reduce propagation, operator routing or server latency,
and may increase power consumption. No reduction is expected where the
normal path already bypasses aggregation.
