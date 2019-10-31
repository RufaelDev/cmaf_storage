# Document name: CMAF Storage Format
## Author: Rufael Mekuria rufael@unified-streaming.com
# CMAF Storage Format 

Common Media Application Track Format (CMAF) [CMAF] has been standardized by ISO/IEC as 23000-19:2018. It defines a segment and track format for streaming content to clients. Common Media Application Track formatted content can be consumed by both DASH and HLS clients. While much industry attention is on the streaming of CMAF content, CMAF can also be interpreted as a base format for storing large collections of content assets. This may especially benefit content that is stored with the objective to later stream it to different device types. This document explores the possibility of using a CMAF Track file-based content storage format for large asset collections. 

## CMAF Media Objects 

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
CMAF resources such as CMAF track files (instead of the file format itself as in MP4). Based on a single set of CMAF resources different manifests can reference different combinations of CMAF resources in a single CMAF presentation. By storing content as CMAF track files, 
combining content in a manifest does not require demultiplexing of content. Combining CMAF track files this way is referred to as late 
binding in CMAF.

## Proposed CMAF Storage Format 
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
             presid1_ssid2_swsid3_rep2_video-avc-1200k.cmf
             presid1_ssid2_swsid4_rep0_video-hevc-1200k.cmfv     // video hevc
             presid1_ssid2_swsid4_rep1_video-hevc-1600k.cmfv     // video hevc
             presid1_ssid3_swsid5_rep0_timed-text-wvtt-en.cmft   // webvtt English 
             presid1_ssid3_swsid6_rep0_timed-text-wvtt-fr.cmft   // webvtt French                      
......
</pre>


_Open question_: would it make sense to be able to annotate the track files themselves, 
allowing the filename/directory structure to be generated based on internal track file annotation ?

## CMAF Storage Format: storage using CMAF track files

The CMAF Storage format will define best practices for storing CMAF content on disk using CMAF track files. 
The example approach in Table 1 and Table 2 can be presented as a guideline with directives for naming the folders and files.
A simple manifest for storing content may be defined, if deemed necessary. 
In addition, annotation of CMAF tracks with metadata may be defined to make it easy to identify the switching set, 
selection set or source content that a CMAF track belongs to from individual track files. 

## CMAF Storage Format: constraints on optional boxes 

The CMAF track files can have optional boxes. 

**sidx**: (segment index): should (must) be present when storing track files.

**prft**: optional, what does this add compared to other times in the trackfile about when the media was created ? 

**emsg**: optional, does it make sense for storing cmaf content ? emsg would need to be duplicated across switching sets, 
typically requirements will be different for different types of event messages.

**styp**: optional, does it make sense for storing cmaf content ? 

**kind**: optional box to signal the role of the track, for example an mpeg dash role urn:mpeg:dash:role:2011 or a w3c html5 role 
  about:html-kind
  
 **sinf**: This optional box can signal the encryption scheme and default encryption by containing schm box, 
 and a schi box containing track encryption box (tenc). **proposal**: store CMAF unencrypted, use storage or transport 
 level encryption instead ? only use common encryption for streaming. **Proposal**: sinf box shall not be present.

## Questions and Answers regarding CMAF Storage Format 
_How can I identify CMAF switching sets from tracks in the CMAF storage format ?_

CMAF defines switching set constraints, 7.3.4 Table 11, if tracks are representing the same content you could identify switching sets implicitly based on the CMAF track format constraints. Tracks with the same source content and same codec fulfilling the switching set constraints can be implicitly derived as being part of the same switching set. 
CMAF storage format may define additional (in-band or out-of-band) signalling to identify switchingset grouping of track files.

_How can I identify selection sets and/or aligned switching sets from CMAF Tracks ?_ 

CMAF does define requirements 7.3.4.4 for aligned switching sets, but these are harder to use for detecting and identifying them, as it is not clear if it makes sense for different codecs. Selection sets may be the default for different switching sets with the same media type (different language subtitles, different video codecs, different audio codecs). CMAF storage format may define additional signalling to identify aligned switching set grouping of track files and selection set grouping of track files.

_How can I identify CMAF tracks based on the same source content ?_

Due to storing tracks in separate files, it can be unclear if tracks are based on identical source content. Typically, the file name could give an identification of the source content. CMAF storage format may define additional content identifiers to be used in track files. Such identifiers could be used to detect the source content.

_How can CMAF stored content be delivered ?_ 

A manifest will be needed to deliver the content. One way to produce the manifest is using a DASH/HLS packager tool. 
Based on CMAF storage format, additional tools may be developed for generating manifests from source content. 
For example, HLS and DASH manifests could be generated automatically for a stored CMAF presentation. Alternatively, 
annotated CMAF tracks can be posted to a publishing point, such as using CMAF ingest by posting individual CMAF fragments [CMAF ingest]. 

[CMAF] ISO/IEC 23000-19:2018
Information technology — Multimedia application format (MPEG-A) — Part 19: Common media application format (CMAF) for segmented media

[CMAF Ingest] DASH-IF Live ingest protocol https://dashif-documents.azurewebsites.net/Ingest/master/DASH-IF-Ingest.html
