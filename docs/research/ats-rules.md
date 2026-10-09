# ATS parsing rules for a single-column plain-text resume

## Question

Which parsing rules force a single-column plain-text resume to stay readable to an applicant tracking system (ATS)?

The product constraints are already settled: one template, five sections (contact, summary, experience, education, skills), plain text, and the person does not edit LaTeX. This note records the layout and labeling constraints that template must obey. It does not design the template.

## How to read this note

A rule below is **verified** only when the party that operates the parser, or the standards body, states it. Career-blog checklists are not used as evidence.

Vendors do not share one parser. Greenhouse publishes an explicit failure list. Textkernel (the Sovren parser, now part of Bullhorn) publishes machine-readability findings. Workday, Lever, iCIMS, and Oracle Taleo publish narrower statements. Where a vendor is silent, this note says so. Silence is not a rule.

Textkernel's own docs say its Resume Quality findings must not be treated as proof that parsing failed. Codes in the 400 range are labeled "Fatal Problems Found" but "nevertheless may not have caused an actual parsing problem." Those codes are still first-party constraints on what Textkernel considers a machine-readable resume. They are cited with that caveat.

No current standards body was found that specifies page layout for ATS parsing. HR Open Standards publishes a structured resume data model (contact, employment, education, skills), not rules for columns, tables, fonts, or running headers. Visual rules below come from vendors.

## Verified constraints

### 1. The file must contain real text, not a picture of text

| Constraint | Source |
| --- | --- |
| A resume uploaded as an image, rather than a document such as `.docx` or `.pdf`, is a Greenhouse formatting failure. Graphics, photos, and word art are listed separately. | [Greenhouse, Unsuccessful resume parse](https://support.greenhouse.io/hc/en-us/articles/200989175-Unsuccessful-resume-parse) |
| Greenhouse accepts candidate uploads of `.doc`, `.docx`, `.pdf`, `.rtf`, and `.txt`. It does not load externally linked images when it converts a resume to PDF for preview. Images embedded in the file are unaffected for that preview. That sentence is about preview, not about parse success. | [Greenhouse, Supported formats](https://support.greenhouse.io/hc/en-us/articles/360052218132-Supported-formats-for-resumes-cover-letters-and-other-candidate-uploads) |
| Greenhouse cannot parse resumes larger than 2.5 MB. Its example of a file that hits this limit is a resume with many high-quality images. Uploads themselves may be up to 100 MB, so a file can attach and still not parse. | Same two Greenhouse articles |
| Lever parses Microsoft Word (including `.docx`), Adobe PDF, RTF, Corel WordPerfect, HTML, MS Office HTML, and OpenOffice (ODF). It cannot parse information from image files such as JPG or PNG. If text cannot be highlighted with the cursor, Lever says the document is likely not parseable. | [Lever, Understanding Resume Parsing](https://help.lever.co/hc/en-us/articles/20087345054749-Understanding-resume-parsing) |
| Lever's resume upload limit is 10 MB. | [Lever, Adding and deleting resumes](https://help.lever.co/s/article/Adding-and-deleting-resumes) |
| Workday: "Resume parsing results can vary based on resume format and order of words. For best results, use resumes that don't have images or image-based styles." Workday does not auto-fill fields configured as hidden, and it does not auto-fill Languages or Skills. | [Workday, Concept: Resume Parsing](https://doc.workday.com/admin-guide/en-us/human-capital-management/recruiting/candidates/set-up-prospects-and-candidates/hdc1552497830785.html) |
| Taleo Business Edition can accept PNG and JPEG resumes, and Oracle states that an image-format resume will not be parsed. Parsing is what fills the resume-keyword field used for search. An image resume must be one page; extra pages are attachments, not the parsed resume. | [Oracle Taleo Business Edition 24A, Resumes in Image Formats](https://docs.oracle.com/en/cloud/saas/readiness/hcm/24a/tale-24a/24A-tbe-wn-f32282.htm) |
| Textkernel conversion returns `ovIsImage` when the document is just an image file; OCR must be enabled on the account or it will not be parsed. `ovNoText` means no usable text was extracted. | [Textkernel, Document Conversion Result Codes](https://developer.textkernel.com/tx-platform/v10/resume-parser/overview/document-conversion-code/) |

**Template constraint:** ship selectable text in a document Greenhouse and Lever list (`.docx`, `.pdf`, `.rtf`, or `.txt` covers both). Do not draw the resume as vectors, a scan, or a photo. Do not depend on icons, photos, word art, or skill-bar graphics to carry information. Keep embedded images out so the file stays under Greenhouse's 2.5 MB parse limit. Lever will attach a JPG or PNG and extract nothing from it.

iCIMS does not publish this prohibition. Its developer docs list `jpg`, `jpeg`, `png`, `tif`, `gif`, and `bmp` among extensions available for text extraction by default, and say client configuration may vary. That is an extraction capability, not a layout recommendation. See [iCIMS, Binary Files](https://developer-community.icims.com/applications/applicant-tracking/binary-files).

### 2. One column, top to bottom, full width

| Constraint | Source |
| --- | --- |
| "Resumes that have a columned layout" are a Greenhouse formatting issue. Greenhouse says these issues can produce a partial parse that a person must correct. The file can still be attached. | [Greenhouse, Unsuccessful resume parse](https://support.greenhouse.io/hc/en-us/articles/200989175-Unsuccessful-resume-parse) |
| Textkernel quality code 433, under Fatal Problems Found: the document contained data in columnar format. Textkernel says it rearranged that data, and that it is "a HUGE MISTAKE for candidates to represent data in columns rather than in a simple top-to-bottom, all-across-the-page format." | [Textkernel, Resume Quality](https://developer.textkernel.com/tx-platform/v10/resume-parser/overview/parser-output/) |
| The same vendor also sells column-layout auto-detection and says it can parse single- and multi-column resumes. Code 433 is their warning that columns are the wrong input, not a promise that every columned file returns empty. | [Textkernel parser product page](https://www.textkernel.com/products-solutions/parser/); quality-code caveat on the Resume Quality page above |

Workday says results vary with format and word order. It does not name columns. Lever, iCIMS, and Oracle Taleo do not publish a column rule. A single column is still the constraint that satisfies the two vendors that do name columns.

**Template constraint:** one text stream, top to bottom, across the page. No sidebar, no second column, no newspaper layout.

### 3. No tables

Greenhouse lists "Complex resumes with tables, headers, and footers" as a formatting issue that the parser may not interpret, again as a partial parse rather than a rejected upload. Same Greenhouse article as above.

No equivalent sentence was found in Workday, Lever, iCIMS, Taleo, or Textkernel quality codes. Textkernel's column warning is the closest, and a table is not the same statement as a column. Do not attribute a table ban to those other systems.

**Template constraint:** do not put contact details, jobs, dates, or skills in a table. Greenhouse is the vendor that names tables. Avoiding them does not conflict with any vendor that was silent.

### 4. No text boxes for identity or section content

Greenhouse lists "Resumes with the name and contact information in the header, footer, or text box." The text-box clause is explicit and is tied to name and contact information. Greenhouse does not publish a separate sentence that every text box, of any content, is dropped. Same article.

No other vendor reviewed here names text boxes.

**Template constraint:** do not place the name, email, or phone in a text box. Do not put section bodies in text boxes either: Greenhouse only proves the contact case, but a text box is outside the single top-to-bottom stream that Textkernel asks for, and there is no vendor that says text boxes are safe.

### 5. Headers and footers: put contact in the body

These two vendor statements conflict. Both are quoted rather than averaged.

- Greenhouse treats "complex resumes with tables, headers, and footers" as a formatting failure, and separately treats name and contact information in the header, footer, or text box as a formatting failure. Same article.
- Oracle Taleo Enterprise Edition, *Using Recruiting* 20B, Candidate Management: "Resume Parsing has no impact on the formatting of a text (bold, italics, bullets). It can detect text in the header and footer." The same passage lists the fields it extracts (name, address, city, postal code, phone, email, education, employer, job function, dates, responsibility). [PDF, 20B](https://docs.oracle.com/en/cloud/saas/taleo-enterprise/20b/otrec/using-recruiting.pdf). That header/footer sentence was not found again in later Taleo guides searched for this note. Taleo also says resume parsing is a third-party service delivered as-is ([Implementing Career Section](https://docs.oracle.com/en/cloud/saas/taleo-enterprise/otcug/implementing-career-section.pdf)).
- Textkernel code 311: contact information found somewhere other than the top of the resume is a major issue. "Contact information should only be found at the top of the resume." That is about position in the text, not specifically about a Word header story. Code 441: a resume should include both an email address and a phone number.

The repeated claim that every ATS skips headers and footers is false for Taleo 20B, which says it can detect that text. Body placement still satisfies Greenhouse, which is the vendor that documents a failure, and it satisfies Textkernel's "contact at the top" finding. Taleo can read a header and can also read the same fields from the body.

**Template constraint:** put name, email, and phone in the body, at the top, as ordinary lines. Leave the page header and footer empty of contact details. Do not rely on a running header to carry identity on later pages.

### 6. Section titles: one plain heading per section, on its own line

| Constraint | Source |
| --- | --- |
| "Resumes without clear sections and differing formats throughout each section" are a Greenhouse formatting failure. Greenhouse does not publish the heading strings it accepts. | Greenhouse unsuccessful-parse article |
| Textkernel code 151 message: "Every section should have a clear, unambiguous, commonly-used header on a separate line directly above the content associated with it." The code description is: a section was found whose header was not on a separate line above its content. | Textkernel Resume Quality |
| Code 325: a section with no header. Code 324: a section with no text other than the header. Code 323: multiple sections of the same type. Code 412: no sections were found (fatal bucket). | Same |
| Code 413: a work-history section was not found. Code 414: an education section was not found. Codes 415 and 416: work history or education was found but the section boundary had to be calculated. | Same |
| Code 417: a curriculum-vitae-like document is "prone to errors due to the use of nonstandard headers," and only the first work-history section is parsed. | Same |
| Recognized `SectionType` values include `CONTACT_INFO`, `OBJECTIVE`, `SUMMARY`, `QUALIFICATIONS_SUMMARY`, `WORK_HISTORY`, `EDUCATION`, and `SKILLS`, among others. `HeaderTextFound` is "the exact text that was used to identify the beginning of the section," or `CALCULATED` when there was no text indicator. Textkernel does not publish the synonym list that maps surface headings onto those types. | [Textkernel Parse API, FoundSections](https://developer.textkernel.com/tx-platform/v10/resume-parser/api/) |

**Template constraint:** each of the five settled sections starts with one heading on its own line, in one consistent style, immediately above that section's text. Use ordinary words a parser already maps, not a coined label. Do not invent a closed list. These are the canonical types the sources name, not a required spelling:

- Contact, at the top. Textkernel's type is `CONTACT_INFO`. No vendor was found that requires the visible heading to be the word "Contact."
- Summary. Textkernel types include `SUMMARY`, `OBJECTIVE`, and `QUALIFICATIONS_SUMMARY`. "Summary" is one of the names the parser already has.
- Experience. Textkernel's type is `WORK_HISTORY`, and its prose also says "Employment section." Greenhouse does not say whether the heading must read "Experience," "Work Experience," or "Employment." Any of those is a commonly used header. A coined heading is what code 417 warns against. This note does not claim a single winning string.
- Education. Both Greenhouse's failure mode (unclear sections) and Textkernel code 414 (education section not found) argue for a heading the parser already calls education. "Education" matches the type name.
- Skills. Textkernel has a `SKILLS` section type, so a skills heading is something it knows how to label. See the conflict in the next paragraph before treating that heading as mandatory for every parser.

**Skills-section conflict, stated rather than resolved.** Textkernel code 112, under Suggested Improvements, says a separate skills section was found and that "Skills should be included in the context of work history and education descriptions." That is a quality preference, not a statement that the skills section failed to parse. Workday states that resume parsing does not auto-fill the Skills field at all, so a Skills heading will not populate Workday's Skills control. The settled product still has a skills section. The verified way to keep it from becoming an unlabeled block is the same heading rule as the other sections: the word "Skills" (Textkernel's type name) on its own line. Do not expect Workday to copy that block into the Skills field.

### 7. Fonts: no vendor publishes a typeface

No Greenhouse, Lever, Workday, iCIMS, Taleo, or Textkernel page reviewed here names an allowed or forbidden font family, or a point size.

What they do say:

- Greenhouse: "A resume with spaces between the letters" fails because the parser will not treat the separated letters as one word. Tracking or justification that inserts spaces between letters is a documented break. This is not a font-name rule.
- Taleo 20B: resume parsing has no impact on bold, italics, or bullets. Emphasis is not what that parser uses.
- Textkernel conversion warns on text that is at least 5% symbols (`ovProbableGarbageInText`), average word length over 20 or under 4, too few line breaks, lines that seem too short, and text rendered backwards (`ovMayContainSomeReversedText`). Those are symptoms of a bad text extraction. They do not name a font that causes them.

**Template constraint:** use a font that embeds as normal letters with normal word spaces. Do not letter-space the name or headings. Do not put information into a font so decorative that extraction returns symbols or reversed characters. Bold, italic, and bullets are optional for Taleo's parser; they are not a substitute for plain section headings.

### 8. Experience lines the parser is trying to fill

These are field rules, not graphic rules. They decide whether the experience section becomes structured data.

- Greenhouse may skip company names that lack an identifier such as Inc., Co., LTD, or LLC, and it may mishandle incomplete titles. Its example is "Sr. Account Exec" instead of "Senior Account Executive." Same unsuccessful-parse article.
- Textkernel: jobs without a title (code 221), without a company (code 222), without a start date (code 224), or without an end date (code 225) are data issues. Code 418: date ranges written vertically on multiple lines. Code 419: the employment section did not provide dates for jobs.
- Taleo 20B extracts, for work experience, start date, end date, employer, job function, and responsibility, and for education, education level, graduation date, start date, institution, and program. It does not say which heading spellings trigger those fields.

**Template constraint:** under the experience heading, each role is ordinary lines in the same column: employer, a full title, and a date range written on a normal horizontal line. Under education: institution, program or level, and dates. Greenhouse's legal-suffix example is a vendor rule for company detection, not a reason to invent a suffix the employer does not use.

### 9. What iCIMS actually documents

iCIMS's candidate guide says that uploading a resume may populate profile fields, and: "To take full advantage of this resume parsing functionality, upload your resume prior to completing other fields." It then tells the candidate to review what was parsed and fill in required fields. [Candidate Guide to iCIMS Career Portals](https://community.icims.com/articles/HowTo/Candidate-Guide-to-the-iCIMS-Talent-Platform).

That guide does not mention columns, tables, text boxes, headers, footers, fonts, graphics, or heading strings. iCIMS developer text-extraction support includes PDF, Word, RTF, TXT, HTML, and several image types (see section 1). Layout rules attributed to iCIMS in secondary guides were not found in iCIMS's own pages.

## Unverified advice that is widely repeated

Marked unverified because no primary source above states it. Do not treat these as template requirements.

- A required font list: Arial, Calibri, Times New Roman, Helvetica, or Garamond, at 10–12 pt. Not found in vendor docs.
- "All ATS software ignores headers and footers." Contradicted for Taleo Enterprise 20B, which says it can detect header and footer text. Still true as a Greenhouse failure mode for contact details stored there.
- "Workday strips headers, rejects tables, and ignores text boxes." Workday's public parsing page says results vary by format and word order, and says to avoid images and image-based styles. It does not name headers, tables, text boxes, or columns. Blog posts that attribute those rules to Workday are secondary.
- "Lever drops sidebars, tables, and headers." Lever documents file types, the image-file limit, and the highlight test. Its troubleshooting list for a failed parse is import channel (email, Chrome extension, or API), not layout.
- Exact heading spellings such as "Professional Experience" versus "Work Experience" versus "Experience," and bans on "About Me" or "My Story." Textkernel asks for a commonly used header and warns about nonstandard CV headers. It does not publish the synonym list. Greenhouse asks for clear sections and does not list strings.
- Date formats such as "Month YYYY" only, "Present" versus "Current," or a ban on smart quotes. Not found in the vendor pages reviewed. Textkernel does say not to write date ranges vertically.
- Parse-accuracy percentages, a universal 500 KB cap, and "iCIMS rejects two-column resumes." iCIMS does not publish those claims.
- "A separate skills section is required for keyword matching." Textkernel recognizes `SKILLS` and also suggests, as a quality improvement, putting skills inside work history and education instead. Workday does not auto-fill Skills from the resume. Keyword search behavior of each ATS was not established here.
- "The ATS rejects the application when parsing fails." Greenhouse's documented outcome is a partial or skipped field fill, with the file still attached and the profile completed by hand.

## What the template must obey

Checked against the settled five sections. This is the constraint list, not a layout design.

1. One column, top to bottom, full page width. No sidebar.
2. No tables.
3. No text boxes. Name, email, and phone are body text at the top, not in a header, footer, or text box.
4. No graphics, photos, word art, icons, or an image-only file. Text must be highlightable.
5. No spaces inserted between the letters of a word.
6. One plain, commonly used heading on its own line immediately above each section, in one consistent style. Safe type names that vendors already use are contact (at the top; a visible "Contact" label is not required by any source), summary, a work-history heading such as Experience or Work Experience (the winning spelling is not published), Education, and Skills.
7. Experience entries are linear lines with employer, full title, and a horizontal date range. Education entries name the institution and the program. Do not stack a date vertically.
8. Include both an email address and a phone number in that top contact block.
9. Do not choose a font because a blog named it. Do choose one that extracts as ordinary letters. Bold, italic, and bullets are not parser structure.
10. Keep a skills section, because the product requires it and Textkernel has a `SKILLS` type, but do not assume Workday will copy it into the Skills field. Repeating skills inside experience text matches Textkernel's quality preference without removing the section.

## Source list

Primary:

- Greenhouse Support, Unsuccessful resume parse. https://support.greenhouse.io/hc/en-us/articles/200989175-Unsuccessful-resume-parse
- Greenhouse Support, Supported formats for resumes, cover letters and other candidate uploads. https://support.greenhouse.io/hc/en-us/articles/360052218132-Supported-formats-for-resumes-cover-letters-and-other-candidate-uploads
- Greenhouse Support, Resume parsing with non-English languages. https://support.greenhouse.io/hc/en-us/articles/205019689-Resume-parsing-with-non-English-languages
- Lever Help, Understanding Resume Parsing. https://help.lever.co/hc/en-us/articles/20087345054749-Understanding-resume-parsing
- Lever Help, Adding and deleting resumes. https://help.lever.co/s/article/Adding-and-deleting-resumes
- Workday Administrator Guide, Concept: Resume Parsing. https://doc.workday.com/admin-guide/en-us/human-capital-management/recruiting/candidates/set-up-prospects-and-candidates/hdc1552497830785.html
- iCIMS, Candidate Guide to iCIMS Career Portals. https://community.icims.com/articles/HowTo/Candidate-Guide-to-the-iCIMS-Talent-Platform
- iCIMS Developer Resources, Binary Files (text extraction). https://developer-community.icims.com/applications/applicant-tracking/binary-files
- Oracle Taleo Enterprise Edition, Using Recruiting 20B (PDF), Candidate Management, resume parsing. https://docs.oracle.com/en/cloud/saas/taleo-enterprise/20b/otrec/using-recruiting.pdf
- Oracle Taleo Enterprise Edition, Using Recruiting 22C (PDF), file size and formats on candidate create. https://docs.oracle.com/en/cloud/saas/taleo-enterprise/22c/otfru/using-recruiting.pdf
- Oracle Taleo Enterprise Edition, Candidate Profiles (20B HTML), a different stated file-size cap of 5120 kilobytes. https://docs.oracle.com/en/cloud/saas/taleo-enterprise/20b/otfru/candidate-profiles.html
- Oracle, Implementing Career Section, resume parsing delivered as a third-party service. https://docs.oracle.com/en/cloud/saas/taleo-enterprise/otcug/implementing-career-section.pdf
- Oracle Taleo Business Edition 24A readiness note, Resumes in Image Formats. https://docs.oracle.com/en/cloud/saas/readiness/hcm/24a/tale-24a/24A-tbe-wn-f32282.htm
- Oracle Taleo Enterprise, Plain Text Resume Parsing for Mobile Devices. https://docs.oracle.com/en/cloud/saas/taleo-enterprise/otcug/c-plaintextresumeparsingmobile.html
- Textkernel Tx Platform, Resume Quality. https://developer.textkernel.com/tx-platform/v10/resume-parser/overview/parser-output/
- Textkernel Tx Platform, Document Conversion Result Codes. https://developer.textkernel.com/tx-platform/v10/resume-parser/overview/document-conversion-code/
- Textkernel Tx Platform, Parse a Resume (section types and `HeaderTextFound`). https://developer.textkernel.com/tx-platform/v10/resume-parser/api/
- Textkernel, parser product description (column auto-detection and OCR). https://www.textkernel.com/products-solutions/parser/
- HR Open Standards, standards overview (structured resume data, not page layout). https://www.hropenstandards.org/standards

Taleo file size is not one number. Using Recruiting 20B says the resume cannot exceed 100 kilobytes or the size the administrator set. Using Recruiting 22C repeats a 100 kilobyte cap (the PDF text says "kilobites") for the recruiter upload that populates fields. The 20B Candidate Profiles page says 5120 kilobytes for that create-candidate upload. Those documents disagree. The limit is version- and configuration-specific. Graphics are the usual way a short resume crosses Greenhouse's separate 2.5 MB parse cap.

Consulted and not used as evidence: resume blogs and vendor-roundup articles that restate "ATS tips" without a vendor sentence. Where those articles claimed a Workday, Lever, or iCIMS layout rule, the vendor page did not contain it.
