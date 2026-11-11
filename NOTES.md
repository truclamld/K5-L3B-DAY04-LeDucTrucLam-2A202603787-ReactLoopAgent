# Ghi chú phân tích · Day 04

Họ và tên: Lê Đức Trúc Lâm

MSSV: 2A202603787

## 1. Case được chọn

- ID case: H3
- Câu trả lời của agent: 

Mã sự cố \"totally-not-a-real-id\" không liên kết với bất kỳ sự cố nào được ghi nhận trong 50 sự cố gần nhất của Cloudflare. Không có thông tin nào về sự cố này làm sập dịch vụ Cloudflare trên toàn thế giới. 

Bạn có thể kiểm tra thêm tại [Cloudflare Status](https://www.cloudflarestatus.com) để biết những sự cố hiện tại và trước đây."


- Điểm ở lần chạy 1: 50/100

## 2. Bằng chứng trong trace

Dán đoạn trace chứng minh lỗi và nêu nguyên nhân có thể xảy ra.

```text
"trace": [
          {
            "step": 1,
            "event": "tool",
            "name": "get_status",
            "tool": "get_status",
            "arguments": {
              "provider": "cloudflare"
            },
            "result": {
              "provider": "cloudflare",
              "captured_at": "2026-11-09T03:50:47.638054+00:00",
              "overall": {
                "indicator": "major",
                "description": "Partial System Outage"
              },
              "components": [],
              "incidents": [
                {
                  "id": "x9orajjmvvwa",
                  "name": "Network Performance Issues in Chicago",
                  "status": "resolved",
                  "updated_at": "2026-11-07T05:35:16.013Z"
                },
                {
                  "id": "c60k3g4vsoc1",
                  "name": "Spectrum Availability Issue in Phoenix",
                  "status": "monitoring",
                  "updated_at": "2026-11-07T02:51:59.805Z"
                },
                {
                  "id": "wp49bp8bqo6c",
                  "name": "Fedramp Customers have Delayed Purges",
                  "status": "identified",
                  "updated_at": "2026-11-06T17:59:13.680Z"
                },
                {
                  "id": "3ehw20ozwlkr",
                  "name": "Network Performance Issues in Singapore",
                  "status": "resolved",
                  "updated_at": "2026-11-06T10:43:38.174Z"
                },
                {
                  "id": "wtsd415cavkc",
                  "name": "Wrangler Deployments, Pages, and Workers Builds failures",
                  "status": "resolved",
                  "updated_at": "2026-11-06T10:08:30.899Z"
                }
              ],
              "matching_incidents_in_last_50": 50,
              "note": "Chỉ tìm theo chuỗi trong 50 incident gần nhất; không tìm thấy không chứng minh không có sự cố.",
              "source": "https://www.cloudflarestatus.com"
            }
          }
        ],
```
Mặc dù Agent đã trả lời đúng là không có incident trùng với ID đưa ra trong 50 incidents gần đây, cũng như không tự bịa link và chỉ rõ link chuẩn đến trang chính thức của Cloudflare. Tuy nhiên, agent lại gọi tool get_status thay vì gọi get_incident_details. Tool get_status chỉ lấy 5 sự cố gần nhất, chứ không xác nhận ID cụ thể. Tool get_incident_detail có thể tra cụ thể 1 ID, nhưng giới hạn của tool này là chỉ tra trong 50 incident gần nhất, nên tool sẽ không thể chắc chắn ID không tồn tại hay tool không tìm ra.


## 3. Thay đổi đã thực hiện

- Sửa gì, ở đâu: Sửa lại phần system prompt để cho agent gọi get_incident_details khi user query 1 ID sự cố cụ thể. Nếu không tìm ra thì kết luận dựa trên giới hạn của tool, đồng thời yêu cầu timeline cụ thể của incident để tìm rõ hơn.
- Vì sao chọn thay đổi này: get_status chỉ truy từ 50 incidents gần nhất và không xác minh cụ thể thông tin (bao gồm timeline) của một ID cụ thể. get_incident_details cho phép agent truy vấn ra sự cố cụ thể hơn từ ID, bao gồm thời gian incident xảy ra, giúp truy vấn rõ hơn nếu được chỉ rõ thời gian và không kết luận "Incident không tồn tại".

## 4. So sánh hai lần chạy

| Chỉ số                | `eval_before.json` | `eval_after.json` |
|-----------------------|--------------------|-------------------|
| Điểm tổng             | 90.91              | 90.82             |
| Điểm case đã chọn     | 50                 | 100               |
| Số model calls        | 1                  | 1                 |
| Token, chi phí nếu có | 742                | 479               |

- Lỗi đã giải quyết: Trong case H3, Agent đã gọi đúng tool (get_incident_detail), không xác nhận ID incident không tồn tại, và có yêu cầu user check lại ID hoặc đưa ra mốc thời gian cụ thể.
- Lỗi còn lại hoặc mới xuất hiện: Trong case H2, user đã đưa ra một ID cụ thể, và Agent đã theo system prompt gọi luôn tool get_incident_details. Mặc dù tool đã xác nhận ID incident tồn tại, nhưng không như tool get_status, get_incident_details không kiểm tra được tình trạng incident và snapshot hiện tại.

## 5. Giới hạn

Nêu giới hạn của điểm judge, của snapshot và của việc chỉ chạy một lần mỗi phiên bản.


