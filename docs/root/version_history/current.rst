1.22.0 (pending)
================

Incompatible Behavior Changes
-----------------------------
*Changes that are expected to cause an incompatibility if applicable; deployment changes are likely required*

* sip-proxy: change API by replacing ``own_domain`` with :ref:`local_services <envoy_v3_api_msg_extensions.filters.network.sip_proxy.v3alpha.LocalService>`.
* tls: set TLS v1.2 as the default minimal version for servers. Users can still explicitly opt-in to 1.0 and 1.1 using :ref:`tls_minimum_protocol_version <envoy_v3_api_field_extensions.transport_sockets.tls.v3.TlsParameters.tls_minimum_protocol_version>`.

Minor Behavior Changes
----------------------
*Changes that may cause incompatibilities for some users, but should not for most*

* access_log: log all header values in the grpc access log.
* config: warning messages for protobuf unknown fields now contain ancestors for easier troubleshooting.
* dynamic_forward_proxy: if a DNS resolution fails, failing immediately with a specific resolution error, rather than finishing up all local filters and failing to select an upstream host.
* ext_authz: added requested server name in ext_authz network filter for auth review.
* file: changed disk based files to truncate files which are not being appended to. This behavioral change can be temporarily reverted by setting runtime guard ``envoy.reloadable_features.append_or_truncate`` to false.
* grpc: flip runtime guard ``envoy.reloadable_features.enable_grpc_async_client_cache`` to be default enabled. async grpc client created through getOrCreateRawAsyncClient will be cached by default.
* health_checker: exposing `initial_metadata` to GrpcHealthCheck in a way similar to `request_headers_to_add` of HttpHealthCheck.
* http: avoiding delay-close for HTTP/1.0 responses framed by connection: close as well as HTTP/1.1 if the request is fully read. This means for responses to such requests, the FIN will be sent immediately after the response. This behavior can be temporarily reverted by setting ``envoy.reloadable_features.skip_delay_close`` to false.  If clients are are seen to be receiving sporadic partial responses and flipping this flag fixes it, please notify the project immediately.
* http: lazy disable downstream connection reading in the HTTP/1 codec to reduce unnecessary system calls. This behavioral change can be temporarily reverted by setting runtime guard ``envoy.reloadable_features.http1_lazy_read_disable`` to false.
* http: now the max concurrent streams of http2 connection can not only be adjusted down according to the SETTINGS frame but also can be adjusted up, of course, it can not exceed the configured upper bounds. This fix is guarded by ``envoy.reloadable_features.http2_allow_capacity_increase_by_settings``.
* http: when writing custom filters, `injectEncodedDataToFilterChain` and `injectDecodedDataToFilterChain` now trigger sending of headers if they were not yet sent due to `StopIteration`. Previously, calling one of the inject functions in that state would trigger an assertion. See issue #19891 for more details.
* listener: the :ref:`ipv4_compat <envoy_api_field_core.SocketAddress.ipv4_compat>` flag can only be set on Ipv6 address and Ipv4-mapped Ipv6 address. A runtime guard is added ``envoy.reloadable_features.strict_check_on_ipv4_compat`` and the default is true.
* perf: ssl contexts are now tracked without scan based garbage collection and greatly improved the performance on secret update.
* router: record upstream request timeouts for all the cases and not just for those requests which are awaiting headers. This behavioral change can be temporarily reverted by setting runtime guard ``envoy.reloadable_features.do_not_await_headers_on_upstream_timeout_to_emit_stats`` to false.
* sip-proxy: add customized affinity support by adding :ref:`tra_service_config <envoy_v3_api_msg_extensions.filters.network.sip_proxy.tra.v3alpha.TraServiceConfig>` and :ref:`customized_affinity <envoy_v3_api_msg_extensions.filters.network.sip_proxy.v3alpha.CustomizedAffinity>`.
* sip-proxy: add support for the ``503`` response code. When there is something wrong occurred, send ``503 Service Unavailable`` back to downstream.
* stateful session http filter: only enable cookie based session state when request path matches the configured cookie path.
* tracing: set tracing error tag for grpc non-ok response code only when it is a upstream error. Client error will not be tagged as a grpc error. This fix is guarded by ``envoy.reloadable_features.update_grpc_response_error_tag``.

Bug Fixes
---------
*Changes expected to improve the state of the world and are unlikely to have negative effects*

* access_log: fix memory leak when reopening an access log fails. Access logs will now try to be reopened on each subsequent flush attempt after a failure.
* data plane: fix crash when internal redirect selects a route configured with direct response or redirect actions.
* data plane: fixing error handling where writing to a socket failed while under the stack of processing. This should genreally affect HTTP/3. This behavioral change can be reverted by setting ``envoy.reloadable_features.allow_upstream_inline_write`` to false.
* eds: fix the eds cluster update by allowing update on the locality of the cluster endpoints. This behavioral change can be temporarily reverted by setting runtime guard ``envoy.reloadable_features.support_locality_update_on_eds_cluster_endpoints`` to false.
* jwt_authn: fixed the crash when a CONNECT request is sent to JWT filter configured with regex match on the Host header.
* tcp_proxy: fix a crash that occurs when configured for :ref:`upstream tunneling <envoy_v3_api_field_extensions.filters.network.tcp_proxy.v3.TcpProxy.tunneling_config>` and the downstream connection disconnects while the the upstream connection or http/2 stream is still being established.
* tls: fix a bug while matching a certificate SAN with an exact value in ``match_typed_subject_alt_names`` of a listener where wildcard ``*`` character is not the only character of the dns label. Example, ``baz*.example.net`` and ``*baz.example.net`` and ``b*z.example.net`` will match ``baz1.example.net`` and ``foobaz.example.net`` and ``buzz.example.net``, respectively.
* upstream: cluster slow start config add ``min_weight_percent`` field to avoid too big EDF deadline which cause slow start endpoints receiving no traffic, default 10%. This fix is releted to `issue#19526 <https://github.com/envoyproxy/envoy/issues/19526>`_.
* upstream: fix stack overflow when a cluster with large number of idle connections is removed.
* xray: fix the AWS X-Ray tracer extension to not sample the trace if ``sampled=`` keyword is not present in the header ``x-amzn-trace-id``.

Removed Config or Runtime
-------------------------
*Normally occurs at the end of the* :ref:`deprecation period <deprecated>`

* access_log: removed ``envoy.reloadable_features.unquote_log_string_values`` and legacy code paths.
* grpc_bridge_filter: removed ``envoy.reloadable_features.grpc_bridge_stats_disabled`` and legacy code paths.
* http: removed ``envoy.reloadable_features.hash_multiple_header_values`` and legacy code paths.
* http: removed ``envoy.reloadable_features.no_chunked_encoding_header_for_304`` and legacy code paths.
* http: removed ``envoy.reloadable_features.preserve_downstream_scheme`` and legacy code paths.
* http: removed ``envoy.reloadable_features.require_strict_1xx_and_204_response_headers`` and ``envoy.reloadable_features.send_strict_1xx_and_204_response_headers`` and legacy code paths.
* http: removed ``envoy.reloadable_features.strip_port_from_connect`` and legacy code paths.
* http: removed ``envoy.reloadable_features.use_observable_cluster_name`` and legacy code paths.
* http: removed ``envoy.reloadable_features.http_transport_failure_reason_in_body`` and legacy code paths.
* http: removed ``envoy.reloadable_features.allow_response_for_timeout`` and legacy code paths.
* http: removed ``envoy.reloadable_features.http2_consume_stream_refused_errors`` and legacy code paths.
* http: removed ``envoy.reloadable_features.internal_redirects_with_body`` and legacy code paths.
* json: removed ``envoy.reloadable_features.remove_legacy_json`` and legacy code paths.
* listener: removed ``envoy.reloadable_features.listener_reuse_port_default_enabled`` and legacy code paths.
* udp: removed ``envoy.reloadable_features.udp_per_event_loop_read_limit`` and legacy code paths.
* upstream: removed ``envoy.reloadable_features.health_check.graceful_goaway_handling`` and legacy code paths.
* xds: removed ``envoy.reloadable_features.vhds_heartbeats`` and legacy code paths.


New Features
------------
* access_log: make consistent access_log format fields ``%(DOWN|DIRECT_DOWN|UP)STREAM_(LOCAL|REMOTE)_*%`` to provide all combinations of local & remote addresses for upstream & downstream connections.
* admin: :http:post:`/logging` now accepts ``/logging?paths=name1:level1,name2:level2,...`` to change multiple log levels at once.
* cluster: support :ref:`override host status restriction <envoy_v3_api_field_config.cluster.v3.Cluster.CommonLbConfig.override_host_status>`.
* config: added new file based xDS configuration via :ref:`path_config_source <envoy_v3_api_field_config.core.v3.ConfigSource.path_config_source>`.
  :ref:`watched_directory <envoy_v3_api_field_config.core.v3.PathConfigSource.watched_directory>` can
  be used to setup an independent watch for when to reload the file path, for example when using
  Kubernetes ConfigMaps to deliver configuration. See the linked documentation for more information.
* config: added new :ref:`custom config validators <config_config_validation>` to dynamically verify config updates.
* cors: add dynamic support for headers ``access-control-allow-methods`` and ``access-control-allow-headers`` in cors.
* http: added random_value_specifier in :ref:`weighted_clusters <envoy_v3_api_field_config.route.v3.RouteAction.weighted_clusters>` to allow random value to be specified from configuration proto.
* http: added support for :ref:`proxy_status_config <envoy_v3_api_field_extensions.filters.network.http_connection_manager.v3.HttpConnectionManager.proxy_status_config>` for configuring `Proxy-Status <https://datatracker.ietf.org/doc/html/draft-ietf-httpbis-proxy-status-08>`_ HTTP response header fields.
* http: make consistent custom header format fields ``%(DOWN|DIRECT_DOWN|UP)STREAM_(LOCAL|REMOTE)_*%`` to provide all combinations of local & remote addresses for upstream & downstream connections.
* http2: re-enabled the HTTP/2 wrapper API. This should be a transparent change that does not affect functionality. Any behavior changes can be reverted by setting the ``envoy.reloadable_features.http2_new_codec_wrapper`` runtime feature to false.
* http3: downstream HTTP/3 support is now GA! Upstream HTTP/3 also GA for specific deployments. See :ref:`here <arch_overview_http3>` for details.
* http3: supports upstream HTTP/3 retries. Automatically retry `0-RTT safe requests <https://www.rfc-editor.org/rfc/rfc7231#section-4.2.1>`_ if they are rejected because they are sent `too early <https://datatracker.ietf.org/doc/html/rfc8470#section-5.2>`_. And automatically retry 0-RTT safe requests if connect attempt fails later on and the cluster is configured with TCP fallback. And add retry on ``http3-post-connect-failure`` policy which allows retry of failed HTTP/3 requests with TCP fallback even after handshake if the cluster is configured with TCP fallback. This feature is guarded by ``envoy.reloadable_features.conn_pool_new_stream_with_early_data_and_http3``.
* local_ratelimit: added support for X-RateLimit-* headers as defined in `draft RFC <https://tools.ietf.org/id/draft-polli-ratelimit-headers-03.html>`_.
* matching: the matching API can now express a match tree that will always match by omitting a matcher at the top level.
* outlier_detection: :ref:`max_ejection_time_jitter<envoy_v3_api_field_config.cluster.v3.OutlierDetection.base_ejection_time>` configuration added to allow adding a random value to the ejection time to prevent 'thundering herd' scenarios. Defaults to 0 so as to not break or change the behavior of existing deployments.
* redis: support for hostnames returned in `cluster slots` response is now available.
* schema_validator_tool: added ``bootstrap`` checking to the
  :ref:`schema validator check tool <install_tools_schema_validator_check_tool>`.
* schema_validator_tool: added ``--fail-on-deprecated`` and ``--fail-on-wip`` to the
  :ref:`schema validator check tool <install_tools_schema_validator_check_tool>` to allow failing
  the check if either deprecated or work-in-progress fields are used.
* schema_validator_tool: fixed linking of all extensions into the
  :ref:`schema validator check tool <install_tools_schema_validator_check_tool>` so that all typed
  configurations can be properly verified.
* schema_validator_tool: the
  :ref:`schema validator check tool <install_tools_schema_validator_check_tool>` will now recurse
  into all sub messages, including Any messages, and perform full validation (deprecation,
  work-in-progress, PGV, etc.). Previously only top-level messages were fully validated.
* stats: histogram_buckets query parameter added to stats endpoint to change histogram output to show buckets.
* tools: the project now ships a :ref:`tools docker image <install_tools>` which contains tools
  useful in support systems such as CI, CD, etc. The
  :ref:`schema validator check tool <install_tools_schema_validator_check_tool>` has been added
  to the tools image.

Deprecated
----------

* config: deprecated :ref:`path <envoy_v3_api_field_config.core.v3.ConfigSource.path>` in favor of
  :ref:`path_config_source <envoy_v3_api_field_config.core.v3.ConfigSource.path_config_source>`
* http: deprecated ``envoy.http.headermap.lazy_map_min_size``.  If you are using this config knob you can revert this temporarily by setting ``envoy.reloadable_features.deprecate_global_ints`` to true but you MUST file an upstream issue to ensure this feature remains available.
* http: removing support for long-deprecated old style filter names, e.g. envoy.router, envoy.lua.
* re2: removed undocumented histograms ``re2.program_size`` and ``re2.exceeded_warn_level``.
