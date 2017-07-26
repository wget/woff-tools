# WOFF File Format  
(draft of 2009-10-03)

Authors:

* Jonathan Kew (Mozilla Corporation)
* Tal Leming (Type Supply)
* Erik van Blokland (LettError)

Copyright ©2009 [Mozilla Corporation](https://www.mozilla.org), [Type Supply](https://www.typesupply.com/), and [LettError](http://letterror.com/). All rights reserved.

* * *

## Version history

[Draft of 2009-10-03](https://web.archive.org/web/20160819054305/http://people.mozilla.org/~jkew/woff/woff-2009-10-03.html)

Corrected editing slip in Font Data Tables section; no change in the actual format specification.

[Draft of 2009-09-16](https://web.archive.org/web/20160819055736/http://people.mozilla.org/~jkew/woff/woff-2009-09-16.html)

First version posted on [http://people.mozilla.org/~jkew/woff/](https://web.archive.org/web/20170630235618/https://people-mozilla.org/~jkew/woff/).

Some earlier drafts, now superseded, were circulated via the www-font mailing list and on [http://www.jfkew.plus.com/woff/](https://web.archive.org/web/20170630235618/https://people-mozilla.org/~jkew/woff/).

* * *

## Introduction

This document specifies a simple compressed file format for fonts, designed primarily for use on the web. The WOFF format is directly based on the table-based sfnt structure used in TrueType[[1]](#fn-1 "footnote 1"), OpenType[[2]](#fn-2 "footnote 2") and Open Font Format[[3]](#fn-3 "footnote 3") fonts, which are collectively referred to as sfnt-based fonts. A WOFF font file is simply a repackaged version of a sfnt-based font in compressed form. The format also allows font metadata and private-use data to be included separately from the font data. WOFF encoding tools convert an existing sfnt-based font into a WOFF formatted file, and user agents restore the original sfnt-based font data for use with a webpage.

In general, the structure and contents of decompressed font data should match that of the original font file. Tools producing WOFF font files may provide other font editing features such as glyph subsetting, validation or font feature additions but these are considered outside the scope of this format. Independent of these features, both tools and user agents must assure that the validity of the underlying font data is preserved.

### Notational Conventions

The key words "must", "must not", "required", "shall", "shall not", "should", "should not", "recommended", "may", and "optional" in this document are to be interpreted as described in RFC 2119[[4]](#fn-4 "footnote 4").

## Overall file structure

The structure of WOFF font files is similar to the structure of sfnt-based fonts: a table directory containing lengths and offsets to individual font data tables, followed by the data tables themselves. The sfnt structure is described fully in the TrueType[[1]](#fn-1 "footnote 1"), OpenType[[2]](#fn-2 "footnote 2") and Open Font Format[[3]](#fn-3 "footnote 3") specifications.

The main body of the file consists of the same collection of tables as the original, uncompressed sfnt-based font, except that each table is individually compressed, and the sfnt table directory is replaced by the WOFF table directory.

A WOFF font file consists of a 44-byte header, followed by a variable-size table directory, a variable number of font tables, an optional block of extended metadata, and an optional block of private data.

<table>
<tr><th colspan="2">WOFF File</th></tr>
<tr><td>WOFFHeader</td><td>File header with basic font type and version, along with offsets to metadata and private data blocks.</td></tr>
<tr><td>TableDirectory</td><td>Directory of font tables, indicating the original size, compressed size and location of each table within the WOFF file.</td></tr>
<tr><td>FontTables</td><td>The font data tables from the original sfnt-based font, compressed to reduce bandwidth requirements.</td></tr>
<tr><td>ExtendedMetadata</td><td>An optional block of extended metadata, represented in XML format and compressed for storage in the WOFF file.</td></tr>
<tr><td>PrivateData</td><td>An optional block of private data for the font designer, foundry, or vendor to use.</td></tr>
</table>

Data values stored in the WOFF Header and WOFF Table Directory sections are stored in big-endian format, just as values are within sfnt-based fonts. The following basic data types are used in the description:

<table>
<tr><th colspan="2">Data types</th></tr>
<tr><td>UInt32</td><td>32-bit (4-byte) unsigned integer in big-endian format</td></tr>
<tr><td>UInt16</td><td>16-bit (2-byte) unsigned integer in big-endian format</td></tr>
</table>

## WOFF Header

The header includes an identifying signature and indicates the specific kind of font data included in the file (TrueType or CFF outline data); it also has a font version number, offsets to additional data chunks, and the number of entries in the table directory that immediately follows the header:

<table>
<tr><th colspan="3">WOFFHeader</th></tr>
<tr><td>UInt32</td><td>signature</td><td>0x774F4646 `wOFF`</td></tr>
<tr><td>UInt32</td><td>flavor</td><td>The "sfnt version" of the original file: 0x00010000 for TrueType flavored fonts or 0x4F54544F `OTTO` for CFF flavored fonts.</td></tr>
<tr><td>UInt32</td><td>length</td><td>Total size of the WOFF file.</td></tr>
<tr><td>UInt16</td><td>numTables</td><td>Number of entries in directory of font tables.</td></tr>
<tr><td>UInt16</td><td>reserved</td><td>Reserved, must be set to zero.</td></tr>
<tr><td>UInt32</td><td>totalSfntSize</td><td>Total size needed for the uncompressed font data, including the sfnt header, directory, and tables.</td></tr>
<tr><td>UInt16</td><td>majorVersion</td><td>Major version of the WOFF font, not necessarily the major version of the original sfnt font.</td></tr>
<tr><td>UInt16</td><td>minorVersion</td><td>Minor version of the WOFF font, not necessarily the minor version of the original sfnt font.</td></tr>
<tr><td>UInt32</td><td>metaOffset</td><td>Offset to metadata block, from beginning of WOFF file; zero if no metadata block is present.</td></tr>
<tr><td>UInt32</td><td>metaLength</td><td>Length of compressed metadata block; zero if no metadata block is present.</td></tr>
<tr><td>UInt32</td><td>metaOrigLength</td><td>Uncompressed size of metadata block; zero if no metadata block is present.</td></tr>
<tr><td>UInt32</td><td>privOffset</td><td>Offset to private data block, from beginning of WOFF file; zero if no private data block is present.</td></tr>
<tr><td>UInt32</td><td>privLength</td><td>Length of private data block; zero if no private data block is present.</td></tr>
</table>

Although only fonts of type 0x00010000 (TrueType) and 0x4F54544F (CFF-based) are widely supported at present, it is not an error in the WOFF file if the `flavor` field contains a different value, indicating a WOFF-packaged version of a different sfnt flavor. (The value 0x74727565 `true` has been used for some TrueType-flavored fonts on Mac OS, for example.) Whether client software will actually support other types of sfnt-based font data is outside the scope of the WOFF specification, which simply describes how the sfnt is repackaged for Web use.

The WOFF `majorVersion` and `minorVersion` fields specify the version number for a given font, which can be based on the version number of the original font but is not required to be. These fields have no effect on font loading or usage behavior in user agents.

The header includes a `reserved` field; this MUST be set to zero. If this field is non-zero, a WOFF processor conforming to this specification MUST reject the file as invalid.

## Table directory

The table directory is an array of WOFF table directory entries, as defined below. The directory follows immediately after the WOFF file header; therefore, there is no explicit offset in the header pointing to this block. Its size is calculated by multiplying the `numTables` value in the WOFF header times the size of a single WOFF table directory. Each table directory entry specifies the size and location of a single font data table.

<table>
<tr><th colspan="3">WOFF TableDirectoryEntry</th></tr>
<tr><td>UInt32</td><td>tag</td><td>4-byte sfnt table identifier.</td></tr>
<tr><td>UInt32</td><td>offset</td><td>Offset to the data, from beginning of WOFF file.</td></tr>
<tr><td>UInt32</td><td>compLength</td><td>Length of the compressed data, excluding padding.</td></tr>
<tr><td>UInt32</td><td>origLength</td><td>Length of the uncompressed table, excluding padding.</td></tr>
<tr><td>UInt32</td><td>origChecksum</td><td>Checksum of the uncompressed table.</td></tr>
</table>

The format of `tag` values are defined by the specifications for sfnt-based fonts. The `offset` and `compLength` fields identify the location of the compressed font table. The `origLength` and `origCheckSum` fields are the length and checksum of the original, uncompressed font table from the table directory of the original font.

The sfnt-based format specifications require that font tables be aligned on 4-byte boundaries. Font tables whose length is not a multiple of 4 are padded with null bytes up to the next 4-byte boundary. WOFF font tables have the same requirement: they MUST begin on 4-byte boundaries and be padded with nulls to the next 4-byte boundary. The `compLength` and `origLength` fields in the header store the exact, unpadded length.

If the length of a compressed font table would be the same as or greater than the length of the original font table, the font table MUST be stored uncompressed in the WOFF file and the `compLength` set equal to the `origLength`. Tools MAY also opt to leave other tables uncompressed (e.g. all tables less than a certain size), in which case `compLength` will be equal to `origLength` for these tables as well. WOFF font files containing table directory entries for which `compLength` is greater than `origLength` are considered invalid and MUST NOT be loaded by user agents. Fonts containing compressed font tables that decompress to a size larger than `origLength` are also considered invalid and MUST NOT be loaded.

The sfnt-based font specifications require that the table directory entries are sorted in ascending order of `tag` value. To simplify processing, WOFF-producing tools MUST produce a table directory with entries in ascending `tag` value order. User agents MUST likewise assure that the sfnt table directory is recreated in ascending `tag` order when restoring the font data to its uncompressed state. The ordering of the font tables themselves is independent of the order of directory entries, as described below.

sfnt-based fonts store a checksum for each table in the table directory, and an overall checksum for the entire font in the `head` table (see the TrueType, OpenType or Open Font Format specifications for the definition of each calculation). Tools producing WOFF fonts MUST validate these checksums, and either correct the values or issue an error message if a discrepancy is found.

In general, WOFF-producing tools generate a WOFF font file with the same set of tables as in the original font. This means that the overall font checksum of a font decompressed from a conformant WOFF file should always match the checksum in the original, valid sfnt-based font file.

In cases where checksum recalculation is necessary or changes to the original font data are made, to subset the glyphs in the font or add special tables for example, conformant tools MUST either remove any digital signature (i.e., a `DSIG` table) or regenerate the signature (if the necessary credentials are available), and MUST correct all affected checksum values and table offsets, both for individual tables and the overall font data checksum contained in the `head` table.

## Font Data Tables

The font data tables in the WOFF file are exactly the same as the tables in the original sfnt-based font file, except that each table MAY have been compressed by the `compress2()` function of zlib[[5]](#fn-5 "footnote 5") (or an equivalent, compatible algorithm). User agents use a function equivalent to the `uncompress` function of zlib[[6]](#fn-6 "footnote 6") to decompress each table. The underlying format these functions use is described in the ZLIB specification[[7]](#fn-7 "footnote 7").

The font data tables MUST be stored immediately following the table directory, without gaps except for the padding that may be required (up to three null bytes at the end of each table) to ensure 4-byte alignment.

Font tables in WOFF files SHOULD be stored in the same order as the original sfnt-based font and user agents SHOULD restore the original uncompressed font table in identical order. The table order is implied by `offset` values in the table directory, sorting table directory entries into ascending `offset` value order produces a list of entries in an order equivalent to that of the font tables. If the table order is not preserved, either during WOFF font creation or during decompression by the user agent or other program, then the font checksum would be incorrect, and MUST be properly updated, and any digital signature recreated or removed.

It is possible that font loading performance in some user agents may be improved by sorting the tables such that general font information, character mapping, and metrics are stored before the actual glyph data. The OpenType and Open Font Format specifications include specific recommendations for different types of fonts. However, such optimizations are an implementation detail and not within the scope of the WOFF specification. But if either a tool or a user agent reorders tables, it MUST recalculate the font checksum in the `head` table, which will be affected by the changed offsets in the sfnt table directory, and remove any `DSIG` table that is invalidated by the changes. A new signature MAY be added to the modified font, as described by the OpenType and Open Font Format specifications (if appropriate signing credentials are available to the tool involved).

## Extended Metadata Block

The WOFF font file MAY include a block of extended metadata, allowing the inclusion of more extensive metadata than is present directly in the original sfnt-based font file. The metadata block consists of XML data compressed by zlib; the file header specifies both the size of the actual compressed and the original uncompressed size in order to facilitate memory allocation.

If present, the metadata MUST be compressed; it is never stored in uncompressed form. If no extended metadata is present, the `metaOffset`, `metaLength` and `metaOrigLength` fields in the WOFF header MUST be set to zero.

The metadata block MUST follow immediately after the last font table. As font tables MUST be padded with null bytes to a 4-byte boundary, the beginning of the metadata block will always be 4-byte aligned. The end of the metadata block is not padded to a 4-byte boundary unless it is followed by a private data block (below).

If the extended metadata is invalid (for example, the offset/length indicate a range outside of the actual WOFF file, or the data cannot be decompressed, or it is not well-formed XML), the WOFF processor MUST proceed as if the metadata block is absent; the font itself remains valid and can still be used (provided its main content of font data tables is valid).

The presence (or absence) and content of the metadata block MUST NOT affect font loading or rendering behavior of user agents; it is intended to be purely informative. However, if user agents provide a means for users to view information about fonts (such as a "Font Information" panel) then they SHOULD treat the metadata block as the primary source, falling back on the font's `name` table entries when relevant extended metadata elements are not present.

[Appendix A](#appendix-a) describes the format for XML metadata[[8]](#fn-8 "footnote 8"), and [appendix B](#appendix-b) illustrates it with a brief example.

## Private Data Block

The WOFF font file MAY include a block of arbitrary data, allowing font creators to include whatever information they wish. The content of this data MUST NOT affect font usage or load behavior. WOFF processors should make no assumptions about the content of a private block; it may (for example) contain ASCII or Unicode text, or some vendor-defined binary data, and it may be compressed or encrypted, but it has no publicly defined format. Conformant user agents will not assume anything about the structure of this data. Only the font developer or vendor responsible for the private block is expected to understand its contents.

The private data block, if present, MUST be the last block in the WOFF file, following all the font tables and any extended metadata block. The private data block MUST begin on a 4-byte boundary in the WOFF file, with up to three null bytes inserted as padding if needed to ensure this. No padding is required at the end of the private data block; any following data does not form part of the WOFF font structure, and MUST be ignored.

If no private data is present, the `privOffset` and `privLength` fields in the WOFF header MUST be set to zero. However, as a conforming WOFF processor does not interpret or even need to access the private data in any way, it will simply ignore these fields. Only a private vendor-specific tool would use them.

* * *

<div id="appendix-a"></div>

## APPENDIX A: Extended Metadata Specification

The extended metadata must be well-formed XML. The use of UTF-8 encoding is recommended.

Several elements store their data in `text` subelements; this is to support localization. The `text` elements may be given a `lang` attribute. The possible values for the `lang` attribute can be found in the IANA Subtag Registry[[9]](#fn-9 "footnote 9"). A user agent displaying metadata is expected to choose a preferred language from among those available, following RFC 4647[[10]](#fn-10 "footnote 10"), and falling back to a default (typically English) or to an unmarked `text` element if no better match is found. Such elements are indicated by the statement "This element may be localized" in the description below; the internal structure of `text` elements with `lang` attributes is not repeated for each element type.

<dl>

<dt>`metadata` element</dt>
<dd>
The main element. This element is required.
<table>
<tr><th colspan="2">attributes</th></tr>
<tr>
<td>version</td>
<td>A version number indicating the format version of the `metadata` element. This is currently `1.0`. This attribute is required.</td>
</tr>
</table>
</dd>

<dt>`uniqueid` element</dt>
<dd>
A unique identifier string for the font. This element is recommended, but not required for the metadata to be valid. This element must be a child of the `metadata` element. This is an empty element.
<table>
<tbody>
<tr><th colspan="2">attributes</th></tr>
<tr>
<td>id</td>
<td>The identification string.</td>
</tr>
</tbody>
</table>
The string defined in the `uniqueid` element is not guaranteed to be truly unique, as there is no central registry or authority to ensure this, but it is intended to allow vendors to reliably identify the exact version of a particular font. The use of "reverse-DNS" prefixes to provide a "namespace" is recommended; this can be augmented by additional identification data of the vendor's own design.
</dd>

<dt>`vendor` element</dt>
<dd>
Information about the font vendor. This element is recommended, but not required for the metadata to be valid. This element must be a child of the `metadata` element. This is an empty element.
<table>
<tbody>
<tr><th colspan="2">attributes</th></tr>
<tr>
<td>name</td>
<td>The name of the font vendor. If the `vendor` element is present in the metadata, this attribute must be present.</td>
</tr>
<tr>
<td>url</td>
<td>The url for the font vendor. This attribute is optional.</td>
</tr>
</tbody>
</table>
</dd>

<dt>`credits` element</dt>
<dd>
Credit information for the font. This can include any type of credit the vendor desires: designer, hinter, and so on. This element is optional. If present, it must be a child of the `metadata` element and it must contain at least one `credit` element. This element has no attributes.
</dd>

<dt>`credit` element</dt>
<dd>
A single credit record. If present, it must be a child of the `credits` element. This is an empty element.
<table>
<tbody>
<tr><th colspan="2">attributes</th></tr>
<tr>
<td>name</td>
<td>The name of the entity being credited. If the `credit` element is present, this attribute must be present.</td>
</tr>
<tr>
<td>url</td>
<td>The url for the entity being credited. This attribute is optional.</td>
</tr>
<tr>
<td>role</td>
<td>The role of the entity being credited. This attribute is optional.</td>
</tr>
</tbody>
</table>
</dd>

<dt>`description` element</dt>
<dd>
An arbitrary text description of the font's design, its history, etc. This element is optional. If present, it must be a child of the `metadata` element. This element may be localized.
</dd>

<dt>`license` element</dt>
<dd>
The license for the font. This element is optional. If present, it must be a child of the `metadata` element. This element may be localized.
<table>
<tbody>
<tr><th colspan="2">attributes</th></tr>
<tr>
<td>url</td>
<td>The url for the license, more information about the license, etc. This attribute is optional.</td>
</tr>
<tr>
<td>id</td>
<td>An identifying string for the license. This attribute is optional.</td>
</tr>
</tbody>
</table>
</dd>

<dt>`copyright` element</dt>
<dd>
The copyright for the font. This element is optional. If present, it must be a child of the `metadata` element. This element may be localized. This element has no attributes.
</dd>
<dt>`trademark` element</dt>
<dd>
The trademark for the font. This element is optional. If present, it must be a child of the `metadata` element. This element may be localized. This element has no attributes.
</dd>

<dt>`licensee` element</dt>
<dd>
The licensee of the font. This element is optional. If present, it must be a child of the `metadata` element. This is an empty element.
<table>
<tbody>
<tr><th colspan="2">attributes</th></tr>
<tr>
<td>name</td>
<td>The name of the licensee. If the `licensee` element is present in the metadata, this attribute must be present.</td>
</tr>
</tbody>
</table>
</dd>

</dl>

* * *

<div id="appendix-b"></div>

## APPENDIX B: Extended Metadata Example

```
<?xml version="1.0" encoding="UTF-8"?>
<metadata version="1.0">
    <uniqueid id="com.example.fontvendor.demofont.rev12345" />
    <vendor name="Font Vendor" url="http://fontvendor.example.com" />
    <credits>
        <credit
            name="Font Designer"
            url="http://fontdesigner.example.com"
            role="Lead" />
        <credit
            name="Another Font Designer"
            url="http://anotherdesigner.example.org"
            role="Contributor" />
        <credit
            name="Yet Another"
            role="Hinting" />
    </credits>
    <description>
        <text lang="en">
            A member of the Demo font family.
            This font is a humanist sans serif style designed
            for optimal legibility in low-resolution environments.
            It can be obtained from fontvendor.example.com.
        </text>
    </description>
    <license url="http://fontvendor.example.com/license"
             id="fontvendor-web-corporate-v2">
        <text lang="en">A license goes here.</text>
        <text lang="fr">Un permis va ici.</text>
    </license>
    <copyright>
        <text lang="en">Copyright ©2009 Font Vendor"</text>
        <text lang="ko">저작권 ©2009 Font Vendor"</text>
    </copyright>
    <trademark>
        <text lang="en">Demo Font is a trademark of Font Vendor</text>
        <text lang="fr">Demo Font est une marque déposée de Font Vendor</text>
        <text lang="de">Demo Font ist ein eingetragenes Warenzeichen der Font Vendor</text>
        <text lang="ja">Demo FontはFont Vendorの商標である</text>
    </trademark>
    <licensee name="Wonderful Websites, Inc." />
</metadata>
```

* * *

## Footnotes

<div id="fn-1"></div>

1. TrueType

   [Apple TrueType specification](https://developer.apple.com/fonts/TrueType-Reference-Manual/). TrueType is a registered trademark of Apple Inc.

<div id="fn-2"></div>

2. OpenType

   [Microsoft OpenType specification](https://www.microsoft.com/en-us/Typography/OpenTypeSpecification.aspx). OpenType is a registered trademark of Microsoft Corporation.

<div id="fn-3"></div>

3. Open Font Format

   [Open Font Format specification (ISO/IEC 14496-22:2009)](http://standards.iso.org/ittf/PubliclyAvailableStandards/c052136_ISO_IEC_14496-22_2009(E).zip).

<div id="fn-4"></div>

4. RFC 2119

   [RFC 2119](https://tools.ietf.org/html/rfc2119) (Key words for use in RFCs to Indicate Requirement Levels)

<div id="fn-5"></div>

5. compress2

   [zlib compress2() function](http://refspecs.linuxbase.org/LSB_3.0.0/LSB-PDA/LSB-PDA/zlib-compress2-1.html)

<div id="fn-6"></div>

6. uncompress

   [zlib uncompress() function](http://refspecs.linuxbase.org/LSB_5.0.0/LSB-Core-generic/LSB-Core-generic/zlib-uncompress-1.html)

<div id="fn-7"></div>

7. ZLIB

   [RFC 1950](https://tools.ietf.org/html/rfc1950) (ZLIB Compressed Data Format Specification)

<div id="fn-8"></div>

8. XML metadata

   It is intended that the XML metadata will not be constrained by a specific DTD, but may be extended by font vendors according to their requirements. However, only metadata elements that conform to a published definition will have the potential to be widely supported (e.g., presented to users on request by a typical user agent).

<div id="fn-9"></div>

9. Language tags

   [IANA Language Subtag Registry](http://www.iana.org/assignments/language-subtag-registry/language-subtag-registry)

<div id="fn-10"></div>

10. RFC 4647

    [RFC 4647](https://tools.ietf.org/html/rfc4647) (Matching of Language Tags)

* * *

