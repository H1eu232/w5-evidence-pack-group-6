# Evidence Pack — W5: The Network Fortress
# Group 6 — HexaCode

---

## 1. Cover

| Field | Details |
|---|---|
| **Group Number** | Group 6 |
| **Member Names** | Minh Tuấn · Thành Vinh · Anh Hoàng · Hoàng Nhân · Mạnh Khang · Ngọc Thắng · Hoàng Thông · Thành Tâm |
| **W5 Repo** | `[[GitHub G6 repo URL]](https://github.com/H1eu232/w5-evidence-pack-group-6.git)` |


---

## 2. Application Carry-Forward Verification

> Trainer verify phần này đầu tiên. Phải có đủ 3 mục bên dưới.

### 2.1 App chạy end-to-end

**Action demo:** `[e.g. User gửi câu hỏi qua chat widget → Lambda xử lý → Bedrock trả lời]`

![App end-to-end](./images/w5-app-end-to-end.png)
<sub>Note: Mô tả request đi qua những bước nào, kết quả trả về là gì.</sub>

---

### 2.2 Architecture Diagram 

![Architecture diagram W5](./images/ArchitectureDiagram.png)

---

### 2.3 W4 Feedback Resolution

**Feedback nhận được từ W4:** `[Trích dẫn 1 feedback cụ thể từ trainer tuần trước]`

**W5 giải quyết thế nào:** `[Mô tả cụ thể đã fix/build thêm gì]`

---

### 2.4 Bedrock Retrieval vẫn hoạt động

![Bedrock retrieval](./images/BedrockRetrieval.png)
<sub>Note: Lambda invoke Bedrock KB, trả về câu trả lời đúng với source citation.</sub>

---

### 2.5 Database Query vẫn hoạt động

![Database query](./images/w5-db-query.png)
<sub>Note: Query chạy được trên RDS, trả về data thật.</sub>

---

## 3. MH1 — Multi-VPC Connectivity

### 3.1 Lựa chọn Path

**Path đã chọn:** `[ ] Path A — VPC Peering` &nbsp;&nbsp; `[ ] Path B — Transit Gateway` &nbsp;&nbsp; `[ ] Path C — Justified Single-VPC`

**Rationale:**
`[Giải thích cụ thể tại sao chọn path này cho ứng dụng — không phải câu chữ chung chung.`
`Nếu Path C: phải giải thích (a) vì sao single-VPC phù hợp, (b) event nào sẽ trigger thêm VPC thứ hai.]`

---

### 3.2 Route Table (cả hai bên nếu Peering / TGW route table nếu TGW)

![Route table](./images/w5-route-table.png)
<sub>Note: Route table show traffic cross-VPC đi đúng target (pcx-XXXX hoặc tgw-XXXX). Giải thích tại sao route này được thêm vào.</sub>

---

### 3.3 Connectivity Test (curl hoặc ping cross-VPC)

```bash
# Paste lệnh test ở đây
# e.g. curl http://10.1.0.x:8080/health  hoặc  ping 10.1.0.x
```

![Connectivity test](./images/w5-connectivity-test.png)
<sub>Note: Output xác nhận traffic cross-VPC đang chảy đúng — response trả về từ instance/service bên VPC kia.</sub>

---

### 3.4 VPC Flow Logs — Bật và có sample entry

![Flow logs enabled](./images/w5-flow-logs-enabled.png)
<sub>Note: VPC Flow Logs đã bật, publish về CloudWatch Logs hoặc S3. Giải thích chọn destination nào và tại sao.</sub>

**Sample Flow Log entry:**

```
[Paste sample log entry ở đây]
# e.g.
# 2 123456789012 eni-xxxx 10.0.1.5 10.1.0.10 443 55234 6 10 840 ACCEPT OK
```

<sub>Note: Giải thích entry này nói lên điều gì — source IP, dest IP, port, action ACCEPT/REJECT.</sub>

---

## 4. MH2 — Network Firewall Hardening

### 4.1 Lựa chọn Path

**Path đã chọn:** `[ ] Path A — AWS Network Firewall` &nbsp;&nbsp; `[ ] Path B — Hardened SG + NACL`

**Rationale:**
`[Nếu Path A: giải thích Lambda/EC2 nào ra internet qua NAT Gateway và tại sao cần firewall.]`
`[Nếu Path B: giải thích (a) vì sao egress firewall không cần cho topology này, (b) traffic nào sẽ đòi deploy nó trong production.]`

---

### 4.2 Cấu hình Firewall / Hardened SG+NACL

**Nếu Path A — Network Firewall:**

![Firewall rule group](./images/w5-firewall-rule-group.png)
<sub>Note: Stateful rule group — domain allowlist hoặc IPS signature. Giải thích rule này block/allow traffic gì và tại sao chọn rule này.</sub>

![Firewall route table](./images/w5-firewall-route-table.png)
<sub>Note: Route table cập nhật — traffic đi qua firewall endpoint trước khi ra NAT Gateway.</sub>

**Nếu Path B — Hardened SG + NACL:**

![SG inbound rules cleaned](./images/w5-sg-cleaned.png)
<sub>Note: Đã xóa mọi rule inbound 0.0.0.0/0 trên port 22/3389. Giải thích thay bằng rule gì.</sub>

![NACL deny rule](./images/w5-nacl-deny.png)
<sub>Note: NACL DENY rule rõ ràng — deny traffic gì, từ đâu, tại sao rule này cần thiết.</sub>

---

### 4.3 Request được cho phép (ACCEPT)

![Request allowed](./images/w5-request-allowed.png)
<sub>Note: Flow Log hoặc Firewall Alert Log cho thấy request hợp lệ được ACCEPT — source, destination, port.</sub>

---

### 4.4 Negative Test — Request bị chặn (DENY/REJECT)

![Request blocked](./images/w5-request-blocked.png)
<sub>Note: Screenshot kết nối bị từ chối — lỗi Connection timed out hoặc Connection refused. Xác nhận firewall/NACL/SG đang ép buộc đúng rule.</sub>

---

## 5. MH3 — File Storage Layer + Backup Plan

### 5.1 EFS File System — Đã tạo và mount

![EFS file system](./images/w5-efs-filesystem.png)
<sub>Note: EFS file system đã tạo, mount target trong private application subnet. Giải thích app dùng EFS để lưu loại file gì (upload chung, session token, model artifact, v.v.).</sub>

![EFS security group](./images/w5-efs-sg.png)
<sub>Note: Security Group của mount target chỉ allow inbound NFS (port 2049) từ SG của app tier — không phải 0.0.0.0/0.</sub>

---

### 5.2 Ghi và đọc file từ EFS mount path

```bash
# Paste lệnh ghi và đọc file ở đây
# e.g.
# echo "test-content" > /mnt/efs/app/testfile.txt
# cat /mnt/efs/app/testfile.txt
```

![EFS write read](./images/w5-efs-write-read.png)
<sub>Note: Chạy trên instance trong private subnet. File ghi vào EFS đọc lại được — xác nhận shared storage hoạt động đúng.</sub>

---

### 5.3 AWS Backup Plan

![Backup plan](./images/w5-backup-plan.png)
<sub>Note: Backup plan có schedule (daily), retention (7+ ngày), backup vault. Bao trùm ít nhất 3 resource: EFS, RDS (W3), EBS (W2).</sub>

![Backup vault](./images/w5-backup-vault.png)
<sub>Note: Backup vault và recovery points đã tạo — ít nhất 1 recovery point Completed.</sub>

---

### 5.4 Restore Test — Bắt buộc

![Restore job completed](./images/w5-restore-completed.png)
<sub>Note: Restore job status: Completed. Ghi lại thời gian restore bắt đầu và kết thúc.</sub>

![Restore data verified](./images/w5-restore-data.png)
<sub>Note: Connect vào resource đã khôi phục, đọc lại data đã biết và xác nhận có ở đó. Đây là bằng chứng backup thực sự chạy được — không chỉ tồn tại.</sub>

---

## 6. MH4 — API Gateway trước Lambda

### 6.1 API Gateway Resource Tree

![API gateway resource](./images/w5-apigw-resource.png)
<sub>Note: Cây resource của API — routes, methods, Lambda Proxy Integration. Giải thích chọn REST API hay HTTP API và tại sao (trade-off auth method).</sub>

---

### 6.2 Throttling — Usage Plan với Rate Limit

![Throttling config](./images/w5-apigw-throttling.png)
<sub>Note: Usage plan cấu hình rate limit (requests/giây) và burst limit. Giải thích chọn giá trị này dựa trên workload ứng dụng.</sub>

---

### 6.3 Authentication Configuration

**Auth method đã chọn:** `[ ] API Key` &nbsp;&nbsp; `[ ] Lambda Authorizer` &nbsp;&nbsp; `[ ] Cognito User Pool Authorizer`

![Auth config](./images/w5-apigw-auth.png)
<sub>Note: Cấu hình auth đang active trên route. Giải thích tại sao chọn method này thay vì các option khác.</sub>

---

### 6.4 Test curl — Authenticated (200)

```bash
# Paste lệnh curl với auth ở đây
# e.g. curl -H "x-api-key: YOUR_KEY" https://xxxx.execute-api.ap-southeast-1.amazonaws.com/prod/chat
```

![curl 200](./images/w5-curl-200.png)
<sub>Note: Request có auth hợp lệ → HTTP 200, response body từ Lambda.</sub>

---

### 6.5 Test curl — Unauthenticated (403)

```bash
# Paste lệnh curl không có auth ở đây
# e.g. curl https://xxxx.execute-api.ap-southeast-1.amazonaws.com/prod/chat
```

![curl 403](./images/w5-curl-403.png)
<sub>Note: Request không có auth → HTTP 403 Forbidden. Xác nhận API Gateway đang enforce auth đúng.</sub>

---

## 7. MH5 — Serverless Scaling Pattern

### 7.1 Pattern đã chọn

**Pattern:** `[ ] Reserved Concurrency` &nbsp;&nbsp; `[ ] Provisioned Concurrency` &nbsp;&nbsp; `[ ] Async + DLQ` &nbsp;&nbsp; `[ ] S3-Event-Triggered Lambda`

**Function áp dụng:** `[Tên Lambda function thật trong ứng dụng — không phải function tạo mới chỉ để làm bài]`

**Rationale:** `[Tại sao pattern này phù hợp với function này trong ứng dụng]`

---

### 7.2 Cấu hình Pattern

**Nếu Reserved Concurrency:**

![Reserved concurrency config](./images/w5-reserved-concurrency.png)
<sub>Note: Max concurrency đã set. Giải thích tại sao function này cần giới hạn — ngăn nuốt hết account limit.</sub>

![Throttle metric](./images/w5-throttle-metric.png)
<sub>Note: CloudWatch metric Throttles hoặc TooManyRequestsException khi invoke vượt limit — xác nhận behavior đang hoạt động đúng.</sub>

---

**Nếu Provisioned Concurrency:**

![Provisioned concurrency config](./images/w5-provisioned-concurrency.png)
<sub>Note: Số lượng provisioned instances và chi phí ước tính/tháng.</sub>

![Cold start before](./images/w5-cold-start-before.png)
<sub>Note: CloudWatch trace TRƯỚC — thấy Init Duration (cold start).</sub>

![Cold start after](./images/w5-cold-start-after.png)
<sub>Note: CloudWatch trace SAU — Init Duration = 0ms, xác nhận pre-warm hoạt động.</sub>

**Estimated cost:** `$[X]/tháng cho [N] provisioned instances`

---

**Nếu Async Invocation + DLQ:**

![Async DLQ config](./images/w5-async-dlq-config.png)
<sub>Note: Lambda cấu hình invocation type Event, DLQ gắn vào (SQS hoặc SNS).</sub>

![DLQ message](./images/w5-dlq-message.png)
<sub>Note: Message rơi vào DLQ kèm chi tiết lỗi — xác nhận failed invocation được capture đúng.</sub>

---

**Nếu S3-Event-Triggered Lambda:**

![S3 event notification](./images/w5-s3-event-notification.png)
<sub>Note: S3 PutObject event notification trên prefix đã cấu hình, trỏ tới Lambda.</sub>

![Lambda cloudwatch log](./images/w5-s3-lambda-log.png)
<sub>Note: CloudWatch log của Lambda sau khi thả file vào S3 — thấy file được đọc, field được trích xuất.</sub>

![DynamoDB output row](./images/w5-dynamodb-output.png)
<sub>Note: Row được ghi vào DynamoDB sau khi Lambda xử lý — xác nhận flow end-to-end hoạt động.</sub>

---

## 8. Negative Security Tests

> Ít nhất một negative test cho mỗi bổ sung W5.

### 8.1 Network / Firewall — Request bị chặn

*(Dùng lại ảnh từ MH2 Section 4.4)*

`[Mô tả attempt: từ đâu, đến đâu, port mấy, kết quả gì]`

---

### 8.2 EFS — Truy cập từ instance không thuộc app-tier SG bị từ chối

![EFS access denied](./images/w5-efs-access-denied.png)
<sub>Note: Mount NFS từ instance ngoài app-tier SG → Connection timed out. Xác nhận EFS mount target SG đang enforce đúng.</sub>

---

### 8.3 API Gateway — Unauthenticated request bị từ chối

*(Dùng lại ảnh từ MH4 Section 6.5)*

`[HTTP 403 Forbidden — API Gateway enforce auth trước khi traffic chạm Lambda]`

---

## 9. Stretch Goals *(Tùy chọn)*

> Chỉ điền nếu đã hoàn tất cả 5 must-haves và Evidence Pack.

### 9.1 `[ ]` VPC Reachability Analyzer

![Reachability analyzer pass](./images/w5-reachability-pass.png)
<sub>Note: Path analysis pass — connectivity đúng.</sub>

![Reachability analyzer fail](./images/w5-reachability-fail.png)
<sub>Note: Sau khi cố ý break route table, Reachability Analyzer phát hiện được lỗi.</sub>

---

### 9.2 `[ ]` Backup Vault Lock

![Vault lock config](./images/w5-vault-lock.png)
<sub>Note: Vault Lock ở Compliance Mode — retention period, không IAM principal nào xóa được recovery point trước khi hết hạn.</sub>

---

### 9.3 `[ ]` Lambda Power Tuning

![Power tuning result](./images/w5-power-tuning.png)
<sub>Note: Kết quả chạy với nhiều mức memory — điểm tối ưu cost-performance là bao nhiêu MB và tại sao.</sub>

---

### 9.4 `[ ]` API Gateway Custom Domain

![Custom domain](./images/w5-custom-domain.png)
<sub>Note: Custom domain với ACM certificate gắn vào stage API Gateway.</sub>

---

### 9.5 `[ ]` CloudFormation Template cho một resource W5

```yaml
# Paste CFN template snippet ở đây
```

![CFN validate output](./images/w5-cfn-validate.png)
<sub>Note: `aws cloudformation validate-template` output — template pass validation.</sub>

---

*— End of W5 Evidence Pack —*
