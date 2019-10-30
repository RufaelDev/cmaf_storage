# Document name: CMAF Storage Format
## Author: Rufael Mekuria rufael@unified-streaming.com
## CMAF Storage Format 

Common Media Application Track Format (CMAF) [CMAF] is standardized by ISO/IEC as 23000-19:2018. It defines a segment and track format for streaming content to clients. Common Media Application Track formatted content can be consumed by both DASH and HLS clients. While much industry attention is on the streaming of CMAF content, the CMAF track file, which is a CMAF track stored as a file, can also be a base format for storing large collections of content assets. This may especially benefit content that is stored with the objective to stream it later to different device types. This document explores the possibility of using a CMAF Track file-based content storage format for large asset collections. 

## CMAF Track formatting constructs

CMAF Media Objects A brief and high-level description of CMAF constructs is given.
For more detailed definitions we refer to ISO/IEC as 23000-19:2018 clause 7.

**CMAF Header**: an ISOBMFF FiletypeBox and MovieBox. CMAF Header can be used as an initialization segment for a DASH representation. 

**CMAF Chunk**: A MovieFragmentBox indexing samples and MovieDataBox containing the samples. 

**CMAF Fragment**: One or more CMAF chunks, starting with an IDR or random-access sample

**CMAF Segment**: One or more CMAF Fragments

**CMAF Track**: A CMAF Header followed by one or more CMAF Fragments

**CMAF Track file**: CMAF Track stored as a file

**CMAF Switching Set**: one or more CMAF Tracks that a client can switch between. The tracks fullfill the switching set constraints defined in CMAF.

**CMAF Aligned Switching Set**: one or more CMAF Switching sets with aligned switching points, the same media type and the same original source content. The tracks fullfill the aligned switching set constraints defined in CMAF.

**CMAF Selection Set**: one or more switching sets containing different aspects of the presentation (i.e. different codec, different language)

**CMAF Presentation**: combination of one or more CMAF switching sets, containing different types of media such as audio, video, subtitles etc. 

## Storing CMAF Media Objects 
The main construct for storing content defined in CMAF is the CMAF track file. 
As CMAF track files are not multiplexed, storing content using CMAF would imply storing each media track in a separate file. 
Late Binding of CMAF Media CMAF is designed in a way that the manifest file can combine different 
CMAF resources such as CMAF track files. Based on a single set of CMAF resources different manifests can reference different combinations 
of CMAF resources in a single CMAF presentation. By storing content as CMAF track files, 
combining content in a manifest does not require de-multiplexing of content. 

## Proposed CMAF Storage Format 
The CMAF storage format stores all content as CMAF track files on disk. The combination of these CMAF tracks should conform to be a CMAF presentation. Table 1 illustrates a possible file storage structure for the storage format.

_Table 1_
<pre>
Root folder
       CMAF_presentation_id_1                      // e.g. batman movie
              CMAF_selection_set_id_1              // e.g. audio
                      CMAF_switching_set_id_1     // e.g. aac encoded audio 
                                      Audio-aac-64k.cmfa
                                      Audio-aac-128k.cmfa
                       CMAF_switching_set_id_2     // e.g. he-aac encoded audio
                                      Audio-he-aac-64k.cmfa 
               CMAF_selection_set_id_2              // e.g. video 
                        CMAF_switching_set_id_3     // e.g. avc encoded video
                                      video-avc-400k.cmfv
                                      video-avc-800k.cmfv
                                      video-avc-1200k.cmfv
                       CMAF_switching_set_id_4     // e.g. hevc encoded video
                                      video-hevc-1200k.cmfv
                                      video-hevc-1600k.cmfv
               CMAF_selection_set_id_3              // e.g. subtitles
                       CMAF_switching_set_id_5     // webvtt English 
                                      timed-text-wvtt-en.cmft
…….                CMAF_switching_set_id_6     // webvtt French
…….                               timed-text-wvtt-fr.cmft
</pre>

## CMAF Storage Format: storage using CMAF track files

The CMAF Storage format will define best practices for storing CMAF content on disk using CMAF track files. 
The example approach in table 1 can be presented as a guideline with directives for naming the folders and files.
A simple manifest for storing content may be defined if deemed necessary. 
In addition, annotation of CMAF tracks with metadata may be defined to make it easy to identify the switchingset, 
selection set or source content that a CMAF track belongs to from individual track files. 

## Questions and Answers regarding CMAF Storage Format 
_How can I identify CMAF switching sets in the CMAF storage format ?_

CMAF defines switching set constraints, 7.3.4 Table 11, if tracks are representing the same content you could identify switching 
sets implicitly based on the CMAF track format constraints.  Tracks with the same source content and same codec fulfilling 
the switching set constraints can be implicitly derived as being part of the same switching set. 
CMAF storage format may define additional  signalling to identify switchingset grouping of track files.

_How can I identify selection sets and/or aligned switching sets in stored CMAF Tracks ?_ 

CMAF does define requirements 7.3.4.4. for aligned switching sets, but these are harder to use for detecting and idenifying them, as it is not clear if it makes sense for different codecs. Selection sets may be the default for different switching sets with the same media type (different language subtitles, different video codecs, different audio codecs). CMAF storafge format may define additional signalling to identify aligned switchingset grouping of track files.

_How can I identify CMAF tracks contain the same source content ?_

Due to storing tracks in separate files, it can be unclear if tracks are based on identical source content. Typically, the file name could give an identification of the source content. CMAF Track file may define content identifiers. CMAF storage format additional signalling to identify source content. 


[CMAF] ISO/IEC 23000-19:2018
Information technology — Multimedia application format (MPEG-A) — Part 19: Common media application format (CMAF) for segmented media
