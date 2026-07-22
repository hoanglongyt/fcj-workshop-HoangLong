---
title: "Blog 3"
date: 2026-06-30
weight: 1
chapter: false
pre: " <b> 3.3. </b> "
---

# AMAZON BEDROCK GUARDRAILS - AN TOÀN VÀ BẢO MẬT CHO ỨNG DỤNG GENERATIVE AI

Amazon Bedrock Guardrails cung cấp giải pháp an toàn đa lớp cho các ứng dụng Generative AI, cho phép doanh nghiệp thiết lập các rào cản bảo vệ (*safeguards*) tùy chỉnh phù hợp với chính sách an toàn thông tin và yêu cầu tuân thủ của tổ chức.

### Các điểm chính cần nắm:

* **Bộ lọc nội dung độc hại (Denied Topics & Content Filters)**: Tự động phát hiện và chặn các câu lệnh (*prompts*) hoặc phản hồi (*responses*) chứa thông tin sai lệch, từ ngữ thù thù, nội dung nhạy cảm hoặc chủ đề bị cấm.
* **Lọc thông tin định danh cá nhân (PII Redaction)**: Tự động nhận diện và làm mờ hoặc ẩn các dữ liệu PII nhạy cảm (như email, số điện thoại, số CCCD, thẻ tín dụng) trong cuộc hội thoại với mô hình AI.
* **Chống tấn công Prompt Injection & Jailbreak**: Tăng cường khả năng phòng thủ trước các kỹ thuật tấn công cố tình vượt rào cản của LLM (*Prompt Injection* / *Jailbreak attacks*).
* **Độc lập với Model**: Guardrails có thể áp dụng nhất quán trên nhiều mô hình nền tảng (*Foundation Models*) khác nhau trong Amazon Bedrock mà không cần điều chỉnh lại cấu hình cho từng mô hình.

---

### Link bài đăng Facebook
- **Đường dẫn bài viết**: [https://www.facebook.com/groups/awsstudygroupfcj/permalink/2220585502039743/#](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2220585502039743/#)

---

### Hướng dẫn chi tiết triển khai Amazon Bedrock Guardrails

#### 1. Tạo Guardrail qua AWS Management Console hoặc AWS CLI
Định nghĩa quy tắc lọc chủ đề cấm (*Denied Topics*) và lọc dữ liệu nhạy cảm (*PII Entities*):

```bash
aws bedrock create-guardrail \
  --name "enterprise-ai-guardrail" \
  --description "Guardrail cho ứng dụng Generative AI doanh nghiệp" \
  --topic-policy-config '{"topicsConfig": [{"name": "FinancialAdvice", "definition": "Cung cấp lời khuyên đầu tư tài chính trực tiếp", "examples": ["Tôi nên mua cổ phiếu nào?"], "type": "DENY"}]}' \
  --sensitive-information-policy-config '{"piiEntitiesConfig": [{"type": "EMAIL", "action": "ANONYMIZE"}, {"type": "PHONE", "action": "ANONYMIZE"}]}' \
  --blocked-input-messaging "Yêu cầu của bạn vi phạm chính sách an toàn nội dung." \
  --blocked-outputs-messaging "Phản hồi đã bị chặn do chứa thông tin không phù hợp."
```

#### 2. Tích hợp Guardrail vào Bedrock Model Invocations
Khi gọi mô hình Bedrock trong ứng dụng (thông qua Python Boto3 SDK), truyền tham số `guardrailIdentifier` và `guardrailVersion`:

```python
import boto3
import json

bedrock = boto3.client('bedrock-runtime', region_name='us-east-1')

response = bedrock.invoke_model(
    modelId='anthropic.claude-3-sonnet-20240229-v1:0',
    guardrailIdentifier='<your-guardrail-id>',
    guardrailVersion='1',
    body=json.dumps({
        "prompt": "Hãy tư vấn cho tôi mã cổ phiếu tốt nhất hôm nay.",
        "max_tokens": 300
    })
)
```

#### 3. Kiểm tra phản hồi đã được lọc
Guardrail sẽ tự động can thiệp và trả về thông báo lỗi quy định sẵn nếu Prompt hoặc Response vi phạm quy tắc an toàn đã thiết lập.