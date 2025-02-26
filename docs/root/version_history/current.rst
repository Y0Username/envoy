1.19.0 (Pending)
================

Incompatible Behavior Changes
-----------------------------
*Changes that are expected to cause an incompatibility if applicable; deployment changes are likely required*

* grpc_bridge_filter: the filter no longer collects grpc stats in favor of the existing grpc stats filter.
  The behavior can be reverted by changing runtime key ``envoy.reloadable_features.grpc_bridge_stats_disabled``.
* tracing: update Apache SkyWalking tracer version to be compatible with 8.4.0 data collect protocol. This change will introduce incompatibility with SkyWalking 8.3.0.

Minor Behavior Changes
----------------------
*Changes that may cause incompatibilities for some users, but should not for most*

* access_log: add new access_log command operator ``%REQUEST_TX_DURATION%``.
* access_log: remove extra quotes on metadata string values. This behavior can be temporarily reverted by setting ``envoy.reloadable_features.unquote_log_string_values`` to false.
* aws_request_signing: requests are now buffered by default to compute signatures which include the
  payload hash, making the filter compatible with most AWS services. Previously, requests were
  never buffered, which only produced correct signatures for requests without a body, or for
  requests to S3, ES or Glacier, which used the literal string ``UNSIGNED-PAYLOAD``. Buffering can
  be now be disabled in favor of using unsigned payloads with compatible services via the new
  `use_unsigned_payload` filter option (default false).
* cluster: added default value of 5 seconds for :ref:`connect_timeout <envoy_v3_api_field_config.cluster.v3.Cluster.connect_timeout>`.
* http: disable the integration between :ref:`ExtensionWithMatcher <envoy_v3_api_msg_extensions.common.matching.v3.ExtensionWithMatcher>`
  and HTTP filters by default to reflects its experimental status. This feature can be enabled by seting
  ``envoy.reloadable_features.experimental_matching_api`` to true.
* http: replaced setting ``envoy.reloadable_features.strict_1xx_and_204_response_headers`` with settings
  ``envoy.reloadable_features.require_strict_1xx_and_204_response_headers``
  (require upstream 1xx or 204 responses to not have Transfer-Encoding or non-zero Content-Length headers) and
  ``envoy.reloadable_features.send_strict_1xx_and_204_response_headers``
  (do not send 1xx or 204 responses with these headers). Both are true by default.
* http: serve HEAD requests from cache.
* http: stop sending the transfer-encoding header for 304. This behavior can be temporarily reverted by setting
  ``envoy.reloadable_features.no_chunked_encoding_header_for_304`` to false.
* http: the behavior of the *present_match* in route header matcher changed. The value of *present_match* is ignored in the past. The new behavior is *present_match* performed when value is true. absent match performed when the value is false. Please reference :ref:`present_match
  <envoy_v3_api_field_config.route.v3.HeaderMatcher.present_match>`.
* listener: respect the :ref:`connection balance config <envoy_v3_api_field_config.listener.v3.Listener.connection_balance_config>`
  defined within the listener where the sockets are redirected to. Clear that field to restore the previous behavior.
* tcp: switched to the new connection pool by default. Any unexpected behavioral changes can be reverted by setting runtime guard ``envoy.reloadable_features.new_tcp_connection_pool`` to false.

Bug Fixes
---------
*Changes expected to improve the state of the world and are unlikely to have negative effects*

* aws_lambda: if `payload_passthrough` is set to ``false``, the downstream response content-type header will now be set from the content-type entry in the JSON response's headers map, if present.
* hot_restart: fix double counting of `server.seconds_until_first_ocsp_response_expiring` and `server.days_until_first_cert_expiring` during hot-restart. This stat was only incorrect until the parent process terminated.
* http: port stripping now works for CONNECT requests, though the port will be restored if the CONNECT request is sent upstream. This behavior can be temporarily reverted by setting ``envoy.reloadable_features.strip_port_from_connect`` to false.
* http: raise max configurable max_request_headers_kb limit to 8192 KiB (8MiB) from 96 KiB in http connection manager.
* listener: fix the crash which could happen when the ongoing filter chain only listener update is followed by the listener removal or full listener update.
* udp: limit each UDP listener to read maxmium 6000 packets per event loop. This behavior can be temporarily reverted by setting ``envoy.reloadable_features.udp_per_event_loop_read_limit`` to false.
* validation: fix an issue that causes TAP sockets to panic during config validation mode.
* xray: fix the default sampling 'rate' for AWS X-Ray tracer extension to be 5% as opposed to 50%.
* zipkin: fix timestamp serializaiton in annotations. A prior bug fix exposed an issue with timestamps being serialized as strings.

Removed Config or Runtime
-------------------------
*Normally occurs at the end of the* :ref:`deprecation period <deprecated>`

* event: removed ``envoy.reloadable_features.activate_timers_next_event_loop`` runtime guard and legacy code path.
* gzip: removed legacy HTTP Gzip filter and runtime guard `envoy.deprecated_features.allow_deprecated_gzip_http_filter`.
* http: removed ``envoy.reloadable_features.allow_500_after_100`` runtime guard and the legacy code path.
* http: removed ``envoy.reloadable_features.always_apply_route_header_rules`` runtime guard and legacy code path.
* http: removed ``envoy.reloadable_features.hcm_stream_error_on_invalid_message`` for disabling closing HTTP/1.1 connections on error. Connection-closing can still be disabled by setting the HTTP/1 configuration :ref:`override_stream_error_on_invalid_http_message <envoy_v3_api_field_config.core.v3.Http1ProtocolOptions.override_stream_error_on_invalid_http_message>`.
* http: removed ``envoy.reloadable_features.http_set_copy_replace_all_headers`` runtime guard and legacy code paths.
* http: removed ``envoy.reloadable_features.overload_manager_disable_keepalive_drain_http2``; Envoy will now always send GOAWAY to HTTP2 downstreams when the :ref:`disable_keepalive <config_overload_manager_overload_actions>` overload action is active.
* http: removed ``envoy.reloadable_features.http_match_on_all_headers`` runtime guard and legacy code paths.
* http: removed ``envoy.reloadable_features.unify_grpc_handling`` runtime guard and legacy code paths.
* tls: removed ``envoy.reloadable_features.tls_use_io_handle_bio`` runtime guard and legacy code path.

New Features
------------

* bandwidth_limit: added new :ref:`HTTP bandwidth limit filter <config_http_filters_bandwidth_limit>`.
* bootstrap: added :ref:`dns_resolution_config <envoy_v3_api_field_config.bootstrap.v3.Bootstrap.dns_resolution_config>` to aggregate all of the DNS resolver configuration in a single message. By setting one such configuration option *no_default_search_domain* as true the DNS resolver will not use the default search domains. And by setting the configuration *resolvers* we can specify the external DNS servers to be used for external DNS query.
* cluster: added :ref:`dns_resolution_config <envoy_v3_api_field_config.cluster.v3.Cluster.dns_resolution_config>` to aggregate all of the DNS resolver configuration in a single message. By setting one such configuration option *no_default_search_domain* as true the DNS resolver will not use the default search domains.
* crash support: restore crash context when continuing to processing requests or responses as a result of an asynchronous callback that invokes a filter directly. This is unlike the call stacks that go through the various network layers, to eventually reach the filter. For a concrete example see: ``Envoy::Extensions::HttpFilters::Cache::CacheFilter::getHeaders`` which posts a callback on the dispatcher that will invoke the filter directly.
* dns resolver: added *DnsResolverOptions* protobuf message to reconcile all of the DNS lookup option flags. By setting the configuration option :ref:`use_tcp_for_dns_lookups <envoy_v3_api_field_config.core.v3.DnsResolverOptions.use_tcp_for_dns_lookups>` as true we can make the underlying dns resolver library to make only TCP queries to the DNS servers and by setting the configuration option :ref:`no_default_search_domain <envoy_v3_api_field_config.core.v3.DnsResolverOptions.no_default_search_domain>` as true the DNS resolver library will not use the default search domains.
* dns resolver: added *DnsResolutionConfig* to combine :ref:`dns_resolver_options <envoy_v3_api_field_config.core.v3.DnsResolutionConfig.dns_resolver_options>` and :ref:`resolvers <envoy_v3_api_field_config.core.v3.DnsResolutionConfig.resolvers>` in a single protobuf message. The field *resolvers* can be specified with a list of DNS resolver addresses. If specified, DNS client library will perform resolution via the underlying DNS resolvers. Otherwise, the default system resolvers (e.g., /etc/resolv.conf) will be used.
* dns_filter: added :ref:`dns_resolution_config <envoy_v3_api_field_extensions.filters.udp.dns_filter.v3alpha.DnsFilterConfig.ClientContextConfig.dns_resolution_config>` to aggregate all of the DNS resolver configuration in a single message. By setting the configuration option *use_tcp_for_dns_lookups* to true we can make dns filter's external resolvers to answer queries using TCP only, by setting the configuration option *no_default_search_domain* as true the DNS resolver will not use the default search domains. And by setting the configuration *resolvers* we can specify the external DNS servers to be used for external DNS query which replaces the pre-existing alpha api field *upstream_resolvers*.
* dynamic_forward_proxy: added :ref:`dns_resolution_config <envoy_v3_api_field_extensions.common.dynamic_forward_proxy.v3.DnsCacheConfig.dns_resolution_config>` option to the DNS cache config in order to aggregate all of the DNS resolver configuration in a single message. By setting one such configuration option *no_default_search_domain* as true the DNS resolver will not use the default search domains. And by setting the configuration *resolvers* we can specify the external DNS servers to be used for external DNS query instead of the system default resolvers.
* http: a new field `is_optional` is added to `extensions.filters.network.http_connection_manager.v3.HttpFilter`. When
  value is `true`, the unsupported http filter will be ignored by envoy. This is also same with unsupported http filter
  in the typed per filter config. For more information, please reference
  :ref:`HttpFilter <envoy_v3_api_field_extensions.filters.network.http_connection_manager.v3.HttpFilter.is_optional>`.
* http: added :ref:`stripping trailing host dot from host header<envoy_v3_api_field_extensions.filters.network.http_connection_manager.v3.HttpConnectionManager.strip_trailing_host_dot>` support.
* http: added support for :ref:`original IP detection extensions<envoy_v3_api_field_extensions.filters.network.http_connection_manager.v3.HttpConnectionManager.original_ip_detection_extensions>`.
  Two initial extensions were added, the :ref:`custom header <envoy_v3_api_msg_extensions.http.original_ip_detection.custom_header.v3.CustomHeaderConfig>` extension and the
  :ref:`xff <envoy_v3_api_msg_extensions.http.original_ip_detection.xff.v3.XffConfig>` extension.
* http: added the ability to :ref:`unescape slash sequences<envoy_v3_api_field_extensions.filters.network.http_connection_manager.v3.HttpConnectionManager.path_with_escaped_slashes_action>` in the path. Requests with unescaped slashes can be proxied, rejected or redirected to the new unescaped path. By default this feature is disabled. The default behavior can be overridden through :ref:`http_connection_manager.path_with_escaped_slashes_action<config_http_conn_man_runtime_path_with_escaped_slashes_action>` runtime variable. This action can be selectively enabled for a portion of requests by setting the :ref:`http_connection_manager.path_with_escaped_slashes_action_sampling<config_http_conn_man_runtime_path_with_escaped_slashes_action_enabled>` runtime variable.
* http: added upstream and downstream alpha HTTP/3 support! See :ref:`quic_options <envoy_v3_api_field_config.listener.v3.UdpListenerConfig.quic_options>` for downstream and the new http3_protocol_options in :ref:`http_protocol_options <envoy_v3_api_msg_extensions.upstreams.http.v3.HttpProtocolOptions>` for upstream HTTP/3.
* jwt_authn: added support to fetch remote jwks asynchronously specified by :ref:`async_fetch <envoy_v3_api_field_extensions.filters.http.jwt_authn.v3.RemoteJwks.async_fetch>`.
* listener: added ability to change an existing listener's address.
* local_rate_limit_filter: added suppoort for locally rate limiting http requests on a per connection basis. This can be enabled by setting the :ref:`local_rate_limit_per_downstream_connection <envoy_v3_api_field_extensions.filters.http.local_ratelimit.v3.LocalRateLimit.local_rate_limit_per_downstream_connection>` field to true.
* metric service: added support for sending metric tags as labels. This can be enabled by setting the :ref:`emit_tags_as_labels <envoy_v3_api_field_config.metrics.v3.MetricsServiceConfig.emit_tags_as_labels>` field to true.
* proxy protocol: added support for generating the header while using the :ref:`HTTP connection manager <config_http_conn_man>`. This is done using the using the :ref:`Proxy Protocol Transport Socket <extension_envoy.transport_sockets.upstream_proxy_protocol>` on upstream clusters.
  This feature is currently affected by a memory leak `issue <https://github.com/envoyproxy/envoy/issues/16682>`_.
* tcp: added support for :ref:`preconnecting <v1.18.0:envoy_v3_api_msg_config.cluster.v3.Cluster.PreconnectPolicy>`. Preconnecting is off by default, but recommended for clusters serving latency-sensitive traffic.
* thrift_proxy: added per upstream metrics within the :ref:`thrift router <envoy_v3_api_msg_extensions.filters.network.thrift_proxy.router.v3.Router>` for request and response size histograms.
* tls: allow dual ECDSA/RSA certs via SDS. Previously, SDS only supported a single certificate per context, and dual cert was only supported via non-SDS.
* udp_proxy: added :ref:`key <envoy_v3_api_msg_extensions.filters.udp.udp_proxy.v3.UdpProxyConfig.HashPolicy>` as another hash policy to support hash based routing on any given key.
* windows container image: added user, EnvoyUser which is part of the Network Configuration Operators group to the container image.

Deprecated
----------

* bootstrap: the field :ref:`use_tcp_for_dns_lookups <envoy_v3_api_field_config.bootstrap.v3.Bootstrap.use_tcp_for_dns_lookups>` is deprecated in favor of :ref:`dns_resolution_config <envoy_v3_api_field_config.bootstrap.v3.Bootstrap.dns_resolution_config>` which aggregates all of the DNS resolver configuration in a single message.
* cluster: the fields :ref:`use_tcp_for_dns_lookups <envoy_v3_api_field_config.cluster.v3.Cluster.use_tcp_for_dns_lookups>` and :ref:`dns_resolvers <envoy_v3_api_field_config.cluster.v3.Cluster.dns_resolvers>` are deprecated in favor of :ref:`dns_resolution_config <envoy_v3_api_field_config.cluster.v3.Cluster.dns_resolution_config>` which aggregates all of the DNS resolver configuration in a single message.
* dynamic_forward_proxy: the field :ref:`use_tcp_for_dns_lookups <envoy_v3_api_field_extensions.common.dynamic_forward_proxy.v3.DnsCacheConfig.use_tcp_for_dns_lookups>` is deprecated in favor of :ref:`dns_resolution_config <envoy_v3_api_field_extensions.common.dynamic_forward_proxy.v3.DnsCacheConfig.dns_resolution_config>` which aggregates all of the DNS resolver configuration in a single message.
* http: :ref:`xff_num_trusted_hops <envoy_v3_api_field_extensions.filters.network.http_connection_manager.v3.HttpConnectionManager.xff_num_trusted_hops>` is deprecated in favor of :ref:`original IP detection extensions<envoy_v3_api_field_extensions.filters.network.http_connection_manager.v3.HttpConnectionManager.original_ip_detection_extensions>`.
