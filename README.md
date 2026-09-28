<p align="center">
  <img src="header.png" alt="Abraxas Labs — hyperswitch-unsigned-webhook" width="100%">
</p>

<p align="center">
  <a href="https://abraxaslabs.tech"><strong>abraxaslabs.tech</strong></a>
  &nbsp;·&nbsp;
  <a href="https://github.com/abraxas">github.com/abraxas</a>
  &nbsp;·&nbsp;
  <a href="https://x.com/abraxas_null">@abraxas_null</a>
  &nbsp;·&nbsp;
  <a href="https://github.com/abraxas/hyperswitch-unsigned-webhook">hyperswitch-unsigned-webhook</a>
</p>

# hyperswitch-unsigned-webhook

**Hyperswitch** `2026.09.21.0` — Juspay

Unpublished Hyperswitch source finding: unsigned Worldpayxml (and BitPay/Shift4) inbound webhooks are treated as verified and consume a payment as Charged without PSync. Worldpayxml verify_webhook_source returns Ok(true). Default algorithm is NoAlgorithm.

| | |
|---|---|
| ID | Unpublished Hyperswitch source finding #1 (no CVE yet) |
| CWE | [CWE-345, CWE-306](https://cwe.mitre.org/data/definitions/306.html) |
| CVSS | **High: 7.5** `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:N/I:H/A:N` |
| Product | [Hyperswitch](https://github.com/juspay/hyperswitch) |
| Affected | all versions **through 2026.09.21.0** (inclusive) |
| Patched | vendor patch — see references |
| Auth | unauthenticated (see source map) |
| License | [GNU Affero GPL v3.0](LICENSE) |
| Lab | `127.0.0.1` only · vendor/client disclosure pack, not a scanner |

---

## Advisory (from the source map)

worldpayxml.rs 1381-1392 Ok(true). incoming.rs 882-889 HandleResponse if source_verified. crypto.rs NoAlgorithm. bitpay.rs/shift4.rs inherit default verify.

---

## Entry

- **Method:** `POST`
- **Path:** `/webhooks/{merchant_id}/worldpayxml`
- **Router:** MerchantIdAuth from path. Worldpayxml verify_webhook_source Ok(true). incoming.rs source_verified HandleResponse. lastEvent SETTLED -&gt; PaymentIntentSuccess -&gt; Charged.
- **Notes:** Unauthenticated unpublished Hyperswitch #1 CWE-345 2026.09.21.0. Needs merchant_id in URL and connector orderCode. Witness: GET /payments/{id} status succeeded. BitPay/Shift4 NoAlgorithm same class. Not eval. Not a reverse shell. Disclose security@juspay.in, not a public GitHub issue.

### Call chain

- `POST /accounts admin_api_key=test_admin`
- `POST /api_keys/{merchant_id}`
- `POST /account/{merchant_id}/connectors worldpayxml`
- `POST /payments confirm=false (then stamp connector_transaction_id + requires_capture)`
- `POST /webhooks/{merchant_id}/worldpayxml unsigned SETTLED XML`
- `GET /payments/{id} status=succeeded`

### Lab preconditions

- Hyperswitch v1 router
- Merchant with worldpayxml MCA
- Payment attempt with connector_transaction_id matching orderCode
- Intent past confirmation (requires_capture / processing)

### Witness

GET /payments/{id} status=succeeded after unsigned SETTLED webhook; attempt Charged

### Not success

- eval/base64/system payload
- reverse shell
- unverified webhooks only PSync
- payment stays requires_confirmation / requires_capture

---

## Patch / remediation

**Do this first:** Apply the vendor patch for **Hyperswitch**. See references.

**Verify after upgrade**

- Re-run `hyperswitch-unsigned-webhook-Abraxas-Labs.py` against the patched build: the mapped witness must **not** appear.
- Confirm the vendor advisory / changeset in the deployed tree (see references).
- A WAF signature is delay, not a patch.

**If you cannot update immediately**

- Disable or isolate the affected component.
- Hunt for the witness condition on production (new privileged users, unexpected files, injected rows — whatever this CVE's map names).

---

## Reproduction (authorized lab)

Target **only** `http://127.0.0.1:18082` (or the loopback you bound). Do not point this script at the internet.

```bash
python3 hyperswitch-unsigned-webhook-Abraxas-Labs.py
```

Success is the **witness** above in the response body. Generic 200 HTML is not it.

---

## Lab images

Loopback stack used to reproduce. Official images unless a `Dockerfile` in this folder builds from source.

- [`lab/docker-compose.yml`](lab/docker-compose.yml)
- [`lab/Dockerfile`](lab/Dockerfile)
- [`lab/run.sh`](lab/run.sh)

`./run.sh` clones Hyperswitch tag **2026.09.21.0** into `lab/hyperswitch-src` and starts `hyperswitch-router:standalone` on loopback `:18082`. Then:

```bash
cd lab
./run.sh
```

Publish nothing except `127.0.0.1`.

---

## References

- [github.com/juspay/hyperswitch](https://github.com/juspay/hyperswitch) tag 2026.09.21.0
- Vendor intake: [security@juspay.in](mailto:security@juspay.in) ([VDP](https://github.com/juspay/hyperswitch/wiki/Vulnerability-Disclosure-Program)). Do **not** open a public GitHub issue.

- Abraxas Labs: [abraxaslabs.tech](https://abraxaslabs.tech) · [github.com/abraxas](https://github.com/abraxas) · [@abraxas_null](https://x.com/abraxas_null)

---

## Records (structured)

```
# Hyperswitch unpublished #1 — unsigned Worldpayxml webhook Charged

CWE: CWE-345, CWE-306
Severity: Critical (HTTP lab SUCCESS, 95%)

## Description

`POST /webhooks/{merchant_id}/worldpayxml` has no HMAC. Worldpayxml `verify_webhook_source` returns `Ok(true)`. `incoming.rs` then `HandleResponse` without PSync. XML `lastEvent=SETTLED` maps to Charged.

## Product

Hyperswitch tag 2026.09.21.0 source; lab image `hyperswitch-router:standalone` v1.127.0. Oracle: payment `status=succeeded` after unsigned SETTLED XML. No Worldpay capture.
```

---

## License

This disclosure pack is licensed under the **GNU Affero General Public License v3.0**. See [LICENSE](LICENSE).

---

## Disclaimer

This pack is for **the vendor, the site owner, and licensed labs**. The script talks to `127.0.0.1`. Using it against systems you do not own is not authorized by Abraxas Labs. No warranty.

<p align="center">
  <a href="https://abraxaslabs.tech">abraxaslabs.tech</a> ·
  <a href="https://github.com/abraxas">github.com/abraxas</a> ·
  <a href="https://x.com/abraxas_null">@abraxas_null</a>
</p>
