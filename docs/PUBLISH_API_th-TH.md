# การเผยแพร่ API

## ภาพรวม

ตัวอย่างนี้รวมคุณสมบัติสำหรับการเผยแพร่ API โดยที่อินเทอร์เฟซแชทอาจสะดวกสำหรับการตรวจสอบเบื้องต้น แต่การนำไปใช้งานจริงขึ้นอยู่กับกรณีการใช้งานเฉพาะและประสบการณ์ผู้ใช้ (UX) ที่ต้องการสำหรับผู้ใช้ปลายทาง ในบางสถานการณ์ UI แบบแชทอาจเป็นตัวเลือกที่เหมาะสม ขณะที่ในกรณีอื่นๆ API แบบสแตนด์อโลนอาจเหมาะสมกว่า หลังจากการตรวจสอบเบื้องต้น ตัวอย่างนี้มีความสามารถในการเผยแพร่บอทที่กำหนดเองตามความต้องการของโครงการ โดยการป้อนการตั้งค่าสำหรับโควตา การจำกัดการใช้งาน แหล่งที่มา ฯลฯ จุดปลายทางสามารถเผยแพร่พร้อมกับคีย์ API ซึ่งให้ความยืดหยุ่นสำหรับตัวเลือกการบูรณาการที่หลากหลาย

## ความปลอดภัย

การใช้เพียงแค่คีย์ API ไม่แนะนำตามที่อธิบายใน: [คู่มือนักพัฒนา AWS API Gateway](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-api-usage-plans.html) ดังนั้น ตัวอย่างนี้จึงใช้การจำกัด IP แอดเดรสอย่างง่ายผ่าน AWS WAF กฎ WAF จะถูกนำไปใช้ร่วมกันทั่วทั้งแอปพลิเคชันเนื่องจากข้อพิจารณาด้านต้นทุน โดยอยู่ภายใต้ข้อสันนิษฐานว่าแหล่งที่มาที่ต้องการจำกัดน่าจะเหมือนกันในทุก API ที่ออก **กรุณาปฏิบัติตามนโยบายความปลอดภัยขององค์กรสำหรับการใช้งานจริง** นอกจากนี้ โปรดดูส่วน[สถาปัตยกรรม](#architecture)

## วิธีเผยแพร่ Bot API แบบกำหนดเอง

### ข้อกำหนดเบื้องต้น

ด้วยเหตุผลทางการกำกับดูแล เฉพาะผู้ใช้ที่จำกัดเท่านั้นที่สามารถเผยแพร่บอทได้ ก่อนการเผยแพร่ ผู้ใช้จะต้องเป็นสมาชิกของกลุ่มที่เรียกว่า `PublishAllowed` ซึ่งสามารถตั้งค่าได้ผ่านคอนโซลการจัดการ > Amazon Cognito User pools หรือ aws cli โปรดทราบว่าสามารถอ้างอิง user pool id ได้โดยเข้าถึง CloudFormation > BedrockChatStack > Outputs > `AuthUserPoolIdxxxx`

![](./imgs/group_membership_publish_allowed.png)

### การตั้งค่าการเผยแพร่ API

หลังจากเข้าสู่ระบบเป็นผู้ใช้ `PublishedAllowed` และสร้างบอท ให้เลือก `API PublishSettings` โปรดทราบว่าสามารถเผยแพร่ได้เฉพาะบอทที่ใช้ร่วมกัน
![](./imgs/bot_api_publish_screenshot.png)

ในหน้าถัดไป คุณสามารถกำหนดค่าพารามิเตอร์ต่างๆ เกี่ยวกับการจำกัดความเร็ว สำหรับรายละเอียดเพิ่มเติม กรุณาดูที่: [Throttle API requests for better throughput](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-request-throttling.html)
![](./imgs/bot_api_publish_screenshot2.png)

หลังการปรับใช้ หน้าจอต่อไปนี้จะปรากฏขึ้นซึ่งคุณสามารถรับ URL endpoint และ API key ได้ คุณยังสามารถเพิ่มและลบ API keys ได้

![](./imgs/bot_api_publish_screenshot3.png)

## สถาปัตยกรรม

API จะถูกเผยแพร่ตามแผนภาพต่อไปนี้:

![](./imgs/published_arch.png)

WAF ใช้สำหรับจำกัดที่อยู่ IP โดยสามารถกำหนดค่าที่อยู่ได้ด้วยการตั้งพารามิเตอร์ `publishedApiAllowedIpV4AddressRanges` และ `publishedApiAllowedIpV6AddressRanges` ใน `cdk.json`

เมื่อผู้ใช้คลิกเผยแพร่บอท [AWS CodeBuild](https://aws.amazon.com/codebuild/) จะเริ่มงานปรับใช้ CDK เพื่อจัดเตรียมสแตก API (ดูเพิ่มเติมที่: [คำนิยาม CDK](../cdk/lib/api-publishment-stack.ts)) ซึ่งประกอบด้วย API Gateway, Lambda และ SQS SQS ใช้เพื่อแยกคำขอผู้ใช้และการดำเนินการ LLM เนื่องจากการสร้างผลลัพธ์อาจใช้เวลาเกิน 30 วินาที ซึ่งเป็นข้อจำกัดของโควตา API Gateway เพื่อดึงผลลัพธ์ จำเป็นต้องเข้าถึง API แบบอะซิงโครนัส สำหรับรายละเอียดเพิ่มเติม ดู [ข้อกำหนด API](#api-specification)

ไคลเอนต์จำเป็นต้องตั้ง `x-api-key` ในส่วนหัวของคำขอ

## ข้อกำหนด API

ดูรายละเอียดได้[ที่นี่](https://aws-samples.github.io/bedrock-claude-chat)