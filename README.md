# Document name: CMAF Storage Format
## Author: Rufael Mekuria rufael@unified-streaming.com
# CMAF Storage Format 

Common Media Application Track Format (CMAF) [CMAF] has been standardized by ISO/IEC as 23000-19:2018. It defines a segment and track format for streaming content to clients. Common Media Application Track formatted content can be consumed by both DASH and HLS clients. While much industry attention is on the streaming of CMAF content, CMAF can also be interpreted as a base format for storing large collections of content assets. This may especially benefit content that is stored with the objective to stream it in a later stage to different device types. 

This document explores the possibility of using a CMAF Track file-based content storage format for large asset collections. One of the key aspects to overcome with this format is the lack of explicit grouping of CMAF tracks and the absence of multiplexing in CMAF track files. 

## Example Workflows and use cases for CMAF Storage format

Example Workflows and use cases using the CMAF storage format include: 

1. Storing large asset collections on disk or in the cloud 
2. Storing assets using a single CMAF source format 
3. On-the-fly packaging or manifest generation of stored CMAF content 
4. Sub-clipping and stitching of CMAF stored content 
5. Searching and identifying content stored on disk or in the cloud
6. Fast and Granular access to content stored on disk or in the cloud

In each of such workflows a simple manifest can be created to identify the location of source content
and the grouping of tracks in CMAF constructs. Other tools could be used to read and process the CMAF 
content for delivery and annotate cmaf tracks with additional information like bit-rate, language and grouping identifiers
if this information was not yet available in the track files. 

The CMAF track file structure itself structure provides fast and granular access by its fragmented file structure and 
the segment index box (sidx).

## CMAF Media Objects Overview

A brief and high-level description of CMAF constructs is given.
For more detailed definitions, refer to ISO/IEC as 23000-19:2018 clause 7.

**CMAF Header**: An ISOBMFF FiletypeBox and MovieBox. CMAF Header can be used as an initialization segment for a DASH representation. See [CMAF] 7.3.2.1 

**CMAF Chunk**: A MovieFragmentBox indexing media samples and MovieDataBox containing the media samples. see [CMAF] 7.3.3.2

**CMAF Fragment**: One or more CMAF chunks, starting with an IDR or random-access sample. see [CMAF] 7.3.2.4

**CMAF Segment**: One or more CMAF Fragments. see [CMAF] 7.3.3.1 

**CMAF Track**: A CMAF Header followed by one or more CMAF Fragments. 7.3.2.2

**CMAF Track file**: CMAF Track stored as a file. See [CMAF] 7.3.2.2

**CMAF Switching Set**: One or more CMAF Tracks that a client can switch between. The tracks fullfill the Switching Set constraints defined in CMAF. see [CMAF] 7.3.4

**CMAF Aligned Switching Set**: One or more CMAF Switching Sets with aligned switching points, the same media type and the same original source content. The tracks fullfill the aligned Switching Set constraints defined in CMAF. see [CMAF] 7.3.

**CMAF Selection Set**: One or more Switching sets all of the same media type, e.g. audio, video, or subtitles. Different Switching sets in a selection set may use different codecs or languages other aspects of a presentation. see [CMAF] 7.3.5

**CMAF Presentation**: Combination of one or more CMAF Switching Sets, containing different types of media such as audio, video, subtitles, etc. 

## Storing CMAF Media Objects 

The main construct for storing content defined in CMAF is the CMAF track file. 
As CMAF track files are not multiplexed, storing content using CMAF would imply storing each media track in a separate file. 
CMAF is designed in a way that the manifest file can combine different 
CMAF resources such as CMAF track files (instead of the file format itself as in MP4 that can multiplex different tracks).

Based on a single set of CMAF resources different manifests can reference different combinations of CMAF resources. 
Combining CMAF track files in this way is referred to as late binding in CMAF.

## CMAF Storage Format: storage using CMAF track files

The CMAF Storage format defines best practices for storing CMAF content on disk using CMAF track files. 

Firstly, the next section adds additional constraints on CMAF Boxes and fields for efficient self contained storage.

Table 1 and Table 2 presents example guidelines for stroring CMAF presentations using file naming and directory naming. 

A simple manifest for storing CMAF content wad defined, as an example. 

In addition, annotation of CMAF tracks with metadata may be needed to make it easy to identify the switching set, 
selection set or source content that a CMAF track belongs to from individual track files without a manifest. 

## CMAF Storage Format: constraints on optional CMAF boxes and fields in tracks

The CMAF track files have optional boxes and fields. For archiving usage of these boxes is recommended in the following ways. 

**sidx**: (segment index): should (must) be present when storing track files.

**prft**: not needed, this does not add much compared to other times in the trackfile about when the media was created.

**emsg**: recommended, consider specific schemes of emsg for inserting metadata or information about the program. emsg will be duplicated cross switching sets. Some emsg may contain program information or metadata or splice point information. Alternatively, 
emsg information could be filtered out and stored in a separate track.

**styp**: optional, styp may be stored in case this benefits your streaming workflow. This box is optional in CMAF

**kind**: optional box to signal the role of the track, for example an mpeg dash role urn:mpeg:dash:role:2011 or a w3c html5 kind role. 

Recommended usage of kind box in udta to signal track role. 

**btrt**: this box can be put in the sample entry (stsd) to store the bit-rate of the track. This information is useful to store as 
it can be used later in manifests for example. 

**elng**: extended language tag may be used to signal the language of the CMAF Track 

**mdhd**: mdhd box should be used to signal the language of the CMAF track

 **sinf**: This optional box can signal the encryption scheme and default encryption by containing schm box, 
 and a schi box containing track encryption box (tenc). **proposal**: store CMAF unencrypted, use storage or transport 
 level encryption instead. Only use common encryption for streaming. **Proposal**: sinf box shall not be present.

## CMAF Storage using track files in a directory and filename structure

The CMAF storage format stores all content as CMAF track files on disk. The combination of these CMAF tracks should conform to be a CMAF presentation. Table 1 illustrates a possible file storage structure for the storage format. Instead of naming based on directory structure, ids could be embedded in the filenames aswell.

_Table 1: storage format using directory structuring_
<pre>
Root folder
       CMAF_presentation_id_1                     // The Batman movie
              CMAF_selection_set_id_1             // audio
                      CMAF_switching_set_id_1     // aac encoded audio 
                                      audio-aac-64k.cmfa
                                      audio-aac-128k.cmfa
                       CMAF_switching_set_id_2    // he-aac encoded audio
                                      audio-he-aac-64k.cmfa 
               CMAF_selection_set_id_2            // video 
                       CMAF_switching_set_id_3    // avc encoded video
                                      video-avc-400k.cmfv
                                      video-avc-800k.cmfv
                                      video-avc-1200k.cmfv
                       CMAF_switching_set_id_4    // hevc encoded video
                                      video-hevc-1200k.cmfv
                                      video-hevc-1600k.cmfv
               CMAF_selection_set_id_3            // subtitles
                       CMAF_switching_set_id_5    // webvtt English 
                                      timed-text-wvtt-en.cmft
                       CMAF_switching_set_id_6    // webvtt French
                                      timed-text-wvtt-fr.cmft
......
</pre>

In the second table the presentation id (presid), switching set id (swsid) and selection set id (ssid) are implicitly coded 
in the filenames instead of the directory structure. In addition the representation numbers are added in the 
filename aswell. 

_Table 2: storage format using naming structuring_
<pre>
Root folder
       CMAF_presentation_id_1                                    // The Batman movie
             presid1_ssid1_swsid1_rep0_audio-aac-64k.cmfa        // aac audio
             presid1_ssid1_swsid1_rep1_audio-aac-128k.cmfa
             presid1_ssid1_swsid2_rep0_audio-he-aac-64k.cmfa     // he-aac audio
             presid1_ssid2_swsid3_rep0_video-avc-400k.cmfv       // video avc
             presid1_ssid2_swsid3_rep1_video-avc-800k.cmfv
             presid1_ssid2_swsid3_rep2_video-avc-1200k.cmfv
             presid1_ssid2_swsid4_rep0_video-hevc-1200k.cmfv     // video hevc
             presid1_ssid2_swsid4_rep1_video-hevc-1600k.cmfv     // video hevc
             presid1_ssid3_swsid5_rep0_timed-text-wvtt-en.cmft   // webvtt English 
             presid1_ssid3_swsid6_rep0_timed-text-wvtt-fr.cmft   // webvtt French                      
......
</pre>


_Open question_: would it make sense to be able to annotate the track files themselves, 
allowing the filename/directory structure to be generated based on internal track file annotation ?

## CMAF Storage using xml schema for CMAF

A possible XML schema for storing the CMAF Stored files could be: 


_Table 3: storage format example xml naming scheme_
```xml
<?xml version="1.0" encoding="utf-8"?>
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema" xmlns="urn:cmaf:storage:2019" targetNamespace="urn:cmaf:storage:2019">
	<xs:annotation>
		<xs:appinfo>CMAF Content Storage Description</xs:appinfo>
		<xs:documentation xml:lang="en">
        This Schema defines a simple description of CMAF stored content by grouping track files
        </xs:documentation>
	</xs:annotation>
    <!-- CMAF stored content -->
	<xs:element name="CMAFStorage" type="CMAFStorageType"/>
	<xs:element name="Presentation" type="PresentationType"/>
       <xs:element name="SelectionSet" type="SelectionSetType"/>
       <xs:element name="SwitchingSetType" type="SwitchingSetType"/>
       <xs:element name="TrackFile" type="TrackFileType"/>
  
	<!-- CMAF XML Storage Type -->
	<xs:complexType name="CMAFStorageType">
		<xs:sequence>
			<xs:element name="Presentation" type="PresentationType" maxOccurs="unbounded"/>
		</xs:sequence>
	</xs:complexType>
	<!-- Presentation -->
	<xs:complexType name="PresentationType">
		<xs:sequence>
			<xs:element name="SelectionSet" type="SelectionSetType" maxOccurs="unbounded"/>
		</xs:sequence>
	</xs:complexType>
	<!-- SelectionSet -->
	<xs:complexType name="SelectionSetType">
		<xs:sequence>
			<xs:element name="SwitchingSet" type="SwitchingSetType" maxOccurs="unbounded"/>
		</xs:sequence>
	</xs:complexType>
	<!-- SwitchingSet -->
	<xs:complexType name="SwitchingSetType">
		<xs:sequence>
			<xs:element name="TrackFile" type="TrackFileType" maxOccurs="unbounded"/>
		</xs:sequence>
	</xs:complexType>
	<!-- Track File -->
	<xs:simpleType name="TrackFileType">
    <xs:restriction base="xs:string">
      <!-- insert characters allowed for file name and location-->
    </xs:restriction>
  </xs:simpleType>
</xs:schema>
```


And the given example from Table 2 would be 

_Table 4 example xml representation of CMAF stored content in Table 2_
```xml
<pre>
<?xml version="1.0" encoding="utf-8"?>
<CMAFStorage
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
 xmlns="urn:cmaf:storage:2019">
  <Presentation>
    <SelectionSet>
      <SwitchingSet>
        <TrackFile>
            presid1_ssid1_swsid1_rep0_audio-aac-64k.cmfa
        </TrackFile>
        <TrackFile>
            presid1_ssid1_swsid1_rep0_audio-aac-128k.cmfa
        </TrackFile>
      </SwitchingSet>
      <SwitchingSet>
        <TrackFile>
            presid1_ssid1_swsid2_rep0_audio-he-aac-64k.cmfa
        </TrackFile>
        </SwitchingSet>
    </SelectionSet>
    <SelectionSet> 
      <SwitchingSet>
        <TrackFile>
            presid1_ssid2_swsid3_rep0_video-avc-400k.cmfv
        </TrackFile>
        <TrackFile>
            presid1_ssid2_swsid3_rep0_video-avc-800k.cmfv
        </TrackFile>
        <TrackFile>
            presid1_ssid2_swsid3_rep0_video-avc-1200k.cmfv
        </TrackFile>
       </SwitchingSet>
       <SwitchingSet>
          <TrackFile>
            presid1_ssid2_swsid4_rep0_video-hevc-1200k.cmfv
          </TrackFile>
          <TrackFile>
             presid1_ssid2_swsid4_rep0_video-hevc-1600k.cmfv
          </TrackFile>
       </SwitchingSet>
     </SelectionSet>
     <SelectionSet>
       <SwitchingSet>
          <TrackFile>
             presid1_ssid3_swsid5_rep0_timed-text-wvtt-en.cmft 
          </TrackFile>
       </SwitchingSet>
       <SwitchingSet>
          <TrackFile>
               presid1_ssid3_swsid5_rep0_timed-text-wvtt-fr.cmft 
          </TrackFile>
       </SwitchingSet>
     </SelectionSet>
    </Presentation>
</CMAFStorage>
```



## Additional Questions and Answers regarding CMAF Track Storage Format in general
_How can I identify CMAF switching sets from tracks in the CMAF Tracks?_

CMAF defines switching set constraints, 7.3.4 Table 11, if tracks are representing the same content you could identify switching sets implicitly based on the CMAF track format constraints. Tracks with the same source content and same codec fulfilling the switching set constraints can be implicitly derived as being part of the same switching set. 
CMAF storage format may define additional (in-band or out-of-band) signalling to identify switchingset grouping of track files.

_How can I identify selection sets and/or aligned switching sets from CMAF Tracks ?_ 

CMAF does define requirements 7.3.4.4 for aligned switching sets, but these are harder to use for detecting and identifying them, as it is not clear if it makes sense for different codecs. Selection sets may be the default for different switching sets with the same media type (different language subtitles, different video codecs, different audio codecs). CMAF storage format may define additional signalling to identify aligned switching set grouping of track files and selection set grouping of track files.

_How can I identify if CMAF tracks are based on the same source content ?_

Due to storing tracks in separate files, it can be unclear if tracks are based on identical source content. Typically, the file name could give an identification of the source content. CMAF storage format may define additional content identifiers to be used in track files. Such identifiers could be used to detect the source content.

_How can CMAF stored content be delivered ?_ 

A manifest will be needed to deliver the content. One way to produce the manifest is using a DASH/HLS packager tool. 
Based on CMAF storage format, additional tools may be developed for generating manifests from source content. 
For example, HLS and DASH manifests could be generated automatically for a stored CMAF presentation. Alternatively, 
annotated CMAF tracks can be posted to a publishing point, such as using CMAF ingest by posting individual CMAF fragments [CMAF ingest]. 

[CMAF] ISO/IEC 23000-19:2018
Information technology — Multimedia application format (MPEG-A) — Part 19: Common media application format (CMAF) for segmented media

[CMAF Ingest] DASH-IF Live ingest protocol https://dashif-documents.azurewebsites.net/Ingest/master/DASH-IF-Ingest.html
