# Security Boundary Schema

## 目标

本文件定义执行面与凭据面的标准安全对象。

它是 [security-boundary.md](security-boundary.md) 的 schema 补充层。

## CredentialReference

```text
CredentialReference
  - credential_id
  - kind
  - scope
  - access_mode
```

## VaultReference

```text
VaultReference
  - vault_id
  - secret_path
  - scope
```

## ResourceBoundAuth

```text
ResourceBoundAuth
  - resource_id
  - auth_type
  - binding_scope
  - redaction_policy
```

## ExecutionRedactionPolicy

```text
ExecutionRedactionPolicy
  - policy_id
  - redact_stdout
  - redact_stderr
  - redact_env
  - allowed_metadata
```

## 规范结论

- 凭据、vault、resource-bound auth 和 execution-side redaction 应使用独立标准对象表达
- sandbox 不应直接拥有原始 credential material
