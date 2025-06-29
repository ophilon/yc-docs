You can convert a dynamic [public IP address](../../vpc/concepts/address.md#public-addresses) to static. Static public IP addresses are reserved and remain attached to respective resources when VMs and network load balancers are stopped.

{% note info %}

Make sure to check out our [pricing policy](../../vpc/pricing.md#prices-public-ip) for inactive static public IP addresses.

{% endnote %}

{% list tabs group=instructions %}

- Management console {#console}

   1. In the [management console]({{ link-console-main }}), select the folder containing the address.
   1. In the list of services, select **{{ ui-key.yacloud.iam.folder.dashboard.label_vpc }}**.
   1. In the left-hand panel, select ![image](../../_assets/console-icons/map-pin.svg) **{{ ui-key.yacloud.vpc.switch_addresses }}**.
   1. Click ![image](../../_assets/console-icons/ellipsis.svg) in the row with the IP address and select **{{ ui-key.yacloud.vpc.addresses.button_action-static }}**.
   1. In the window that opens, click **{{ ui-key.yacloud.vpc.addresses.popup-confirm_button_static }}**.

- CLI {#cli}

   {% include [include](../../_includes/cli-install.md) %}

   {% include [default-catalogue](../../_includes/default-catalogue.md) %}

   1. See the description of the CLI commands for updating the address attributes:

      ```bash
      yc vpc address update --help
      ```

   1. Get a list of addresses in the default folder:

      ```bash
      yc vpc address list
      ```

      Result:

      ```text
      +----------------------+------+---------------+----------+------+
      |          ID          | NAME |    ADDRESS    | RESERVED | USED |
      +----------------------+------+---------------+----------+------+
      | e2l46k8conff******** |      | 84.201.177.41 | false    | true |
      +----------------------+------+---------------+----------+------+
      ```

      The `false` value of the `reserved` parameter for the IP address with the `e2l46k8conff********` ID shows that this address is dynamic.

   1. Convert it to static by using the `--reserved=true` parameter and the address ID:

      ```bash
      yc vpc address update --reserved=true e2l46k8conff********
      ```

      Result:

      ```text
      id: e2l46k8conff********
      folder_id: b1g7gvsi89m3********
      created_at: "2021-01-14T09:36:46Z"
      external_ipv4_address:
        address: 84.201.177.41
        zone_id: {{ region-id }}-a
        requirements: {}
      reserved: true
      used: true
      ```

      Now that the `reserved` parameter is `true`, the IP address is static.

- API {#api}

  To change the type of a public IP address from dynamic to static, use the [update](../../vpc/api-ref/Address/update.md) REST API method for the [Address](../../vpc/api-ref/Address/index.md) resource or the [AddressService/Update](../../vpc/api-ref/grpc/Address/update.md) gRPC API call, and provide the following in the request:

  * ID of the IP address you want to convert to static in the `addressId` parameter.

    {% include [get-address-id](../../_includes/vpc/get-adress-id.md) %}

    {% include [get-catalog-id](../../_includes/get-catalog-id.md) %}

  * `true` in the `reserved` parameter.
  * Name of the `reserved` parameter in the `updateMask` parameter.

  {% include [Note API updateMask](../../_includes/note-api-updatemask.md) %}

{% endlist %}

The number of static public IP addresses is [limited](../../vpc/concepts/limits.md#vpc-quotas). If the number allowed by the quota is not enough for you, contact [support]({{ link-console-support }}) to have your quota increased.
