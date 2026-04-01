# Skills.md

## Skill Name
masterpiece-clone

---

## Description
Execute the **MasterPiece Clone** workflow to process image inputs through prompt-to-text and image-to-image generation nodes.

This workflow is designed for high-quality image manipulation and enhancement, leveraging multiple image inputs and a guided text prompt to create unique visual masterpieces.

---

## Service Overview

- 🌐 **Product Website / Console**:  
  https://ai.ngmob.com  

- 🔗 **API Base URL**:  
  https://api.ngmob.com  

---

## Use Cases

- Image content analysis and description
- Automated image tagging and captioning
- Visual data extraction
- Image style transfer and enhancement
- Photo editing and retouching automation
- Visual content creation for marketing

---

## Inputs

| Name | Type | Required | Description |
|------|------|----------|-------------|
| Image Input | string (URL) | ✅ | Publicly accessible first image URL |
| Image Input 1 | string (URL) | ✅ | Publicly accessible second image URL |
| Text Input | string | ✅ | Text prompt or content |

---

## Preview

![MasterPiece](https://ai.ngmob.com/assets/img/hero-img.png)

### Example (Recommended)

```json
{
  "Image Input": "https://example.com/image1.jpg",
  "Image Input 1": "https://example.com/image2.jpg",
  "Text Input": "a cinematic portrait"
}
```
