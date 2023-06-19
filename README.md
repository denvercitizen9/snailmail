#snailmail
Entry for [So You Think You Can Hack](https://devpost.com/software/snail-mail-sj091u)

###Step 1: Redirect Mail
We would work with a 3rd party to redirect your physical mail to a facility that digitizes it.  You never touch it.
In this repo you will see an example of this: `SampleCableBill`

###Step 2: Use fancy OSS AI-based OCR to view the dark data
Since this is an Open Source focus we want to use [PaddlePaddle's OCRv3, which you can see live here](https://www.paddlepaddle.org.cn/hub/scene/ocr').  We didn't get as far as hosting this ourselves, but one could imagine how that would happen.  Using the SampleCableBill.pdf, the complete response that we get from PP-OCR is provided in `pp_ocrv3.json`   

Since we can't use that as-is, we simplify it in a way that is more digestable.  The following Powershell code is one example of how to do this:
```
$jsonContent = Get-Content -Raw -Path "pp_ocrv3.json"
$jsonObject = ConvertFrom-Json $jsonContent
$textFields = $jsonObject.data | Select-Object -ExpandProperty text
$textFields
```  
###Step 3: Summarize Mail Content

Now that we have all of the text Fields available we can provide them to ChatGPT (or a superior LLM like Dolly) with the `gpt_prompt.txt` followed by `<textFields>` in the prompt.  ChatGPT will respond with something like the following

``` {
  "input": "Billing statement.json",
  "sender": "Cable Company",
  "title": "Billing Statement",
  "summary": "Total Amount Due: $131.12 | Due Date: Jan 30, 2012. The billing statement from the Cable Company includes the total amount due of $131.12, with a due date of Jan 30, 2012. It requires attention and payment, providing essential financial information.",
  "tinySummary": "Amount Due: $131.12 | Due: Jan 30",
  "category": "Semi-Important",
  "explanation": "The document is categorized as Semi-Important because it contains a billing statement with the total amount due and due date. It provides financial information requiring attention but does not involve urgent matters or legal documents."  
} ```       
   
Note: eventual plan would be full OSS and self-hosted, since we don't really want to be sending everyone's mail to a 3rd party.  
   
###Step 4: Send Email Digest of your Physical Mail

With the content identified it's now possible to repeat for a set of documents (e.g., all mail received this day/week/etc.) and send the entire collection together in one summary.