---
title: 'Bytesize 23: nf-core/hic'
subtitle: Nicolas Servant - Institut Curie, France
type: talk
startDate: '2021-10-12'
startTime: '13:00+02:00'
endDate: '2021-10-12'
endTime: '13:30+02:00'
embedAt: 'hic'
youtubeEmbed: https://youtu.be/AFHaX_GRNBU
locationURL:
  - https://youtu.be/AFHaX_GRNBU
  - https://doi.org/10.6084/m9.figshare.16795882.v1
  - https://www.bilibili.com/video/BV1Uh411J7BV/
---

# nf-core/bytesize

Join us for a special pipeline-focused episode of our **weekly series** of short talks: **“nf-core/bytesize”**.

Just **15 minutes** + questions, we will be focussing on topics about using and developing nf-core pipelines.
These will be recorded and made available at <https://nf-co.re>
It is our hope that these talks / videos will build an archive of training material that can complement our documentation. Got an idea for a talk? Let us know on the [`#bytesize`](https://nfcore.slack.com/channels/bytesize) Slack channel!

## Bytesize 23: nf-core/hic

This week, Nicolas Servant ([@nservant](https://github.com/nservant/)) will tell us all about the nf-core/hic pipeline.

[0:00](https://youtu.be/AFHaX_GRNBU?list=PL3xpfTVZLcNiSvvPWORbO32S1WDJqKp1e&t=0) Hi everyone. Happy to have you here again for our bytesize talk today. We are joined today by Nicolas Servant, who is based at the Institute Curie in Paris, France. He is talking to us about the nf-core/hic pipeline. If you have any questions for Nicolas during the talk, please put them in the chat or even unmute yourself at the end of the session and ask them directly. We started doing that from today's talk. Just to let you know, this video and all previous bytesize talks are on our YouTube channel, so you can feel free to consult them there as well. All links are still on our homepage. With that, thanks for joining us again Nicolas and over to you.

[0:42](https://youtu.be/AFHaX_GRNBU?list=PL3xpfTVZLcNiSvvPWORbO32S1WDJqKp1e&t=42) Thank you very much.

[0:49](https://youtu.be/AFHaX_GRNBU?list=PL3xpfTVZLcNiSvvPWORbO32S1WDJqKp1e&t=49) So I’ll introduce you to the project by first providing some context on chromatin organisation, then I will describe the standard HiC data analysis workflow, talk about the nf-core/hic pipeline, and finally discuss its future development.

[1:15](https://youtu.be/AFHaX_GRNBU?list=PL3xpfTVZLcNiSvvPWORbO32S1WDJqKp1e&t=75) Now for a broad introduction to chromatin organisation. What we know so far is that there are different levels of chromatin organisation within the nucleus. The first is chromosome territories that were described a long time ago using microscopy; each chromosome occupies its own space within the nucleus. Then zooming in at the level of the chromosome, there are broadly two main types of chromosome compartments: one which is the open and active chromatin that is transcribed and regulated, and the other which is more compact and inactive. There are also other subtypes of chromatin compartments, but today we will focus on the active and inactive types. So zooming in further, we see what are called TADs or Topological Associated Domains that are subunits of regulation. Within these domains are all the regulatory elements that can contact genes and promote transcription. However, there are almost no contacts between TADs, there are interdomain proteins such as CTCF that impede their interaction. Zooming into the TADs, we see loops between regulatory elements and genes. So these four levels of chromatin organisation form the basis of different biological questions. For example, changes during cell differentiation, embryonic development, investigating the link between gene regulation and chromatin organisation etc. So that was the background, so let’s look at how one can access this information.

[3:57](https://youtu.be/AFHaX_GRNBU?list=PL3xpfTVZLcNiSvvPWORbO32S1WDJqKp1e&t=237) The HiC protocol was first published in 2009. This slide presents the main steps of the protocol. Very briefly, DNA is crosslinked, the adduct is cut using a restriction enzyme, the ends are ligated which is followed by pull-downs of the ligated products, and finally these are sequenced using paired-end sequencing. This gives these contact maps that can be at different resolutions, the higher the resolution, the higher the sequencing depth, upto almost a billion reads per sample. The one on the right hand side of the slide at the very top is an example of a genome-wide map - each square represents a chromosome.

[5:00](https://youtu.be/AFHaX_GRNBU?list=PL3xpfTVZLcNiSvvPWORbO32S1WDJqKp1e&t=300) So let’s try and link HiC with the different levels of chromatin organisation. The slide shows chromatin territories that correspond to the map on the top left. Each chromosome can be seen to be in contact with itself but not with other chromosomes. Then let’s move to the map on the top right, which shows what is happening at the level of an individual chromosome. You see this checked pattern which correlates with the open and closed compartments. Further zooming in (map on the bottom right of the slide), you see TADs represented by triangles, and finally in the map on the bottom left, you see direct loops and contacts between the loci and the genome.

[5:52](https://youtu.be/AFHaX_GRNBU?list=PL3xpfTVZLcNiSvvPWORbO32S1WDJqKp1e&t=352) Let’s get back to the methods: fixation, digestion, ligation, enrichment, and sequencing. There have been different protocols over the years, such as _in situ_ HiC, single cell HiC, DNase HiC where DNase is used for digestion, and then there are several commercially available kits.

[6:51](https://youtu.be/AFHaX_GRNBU?list=PL3xpfTVZLcNiSvvPWORbO32S1WDJqKp1e&t=411) Now let’s move on to HiC data analysis. There are three main steps. The first is read mapping, where the first read represents the first interactor and the second read represents the second interactor, and these reads are to be aligned to the reference genome. You might need to deal with chimeric reads especially if one of the reads in a pair spans a ligation junction, which means that both interactors are present in one of the reads. So one usually would just map the 5’ end of the read. Another thing that is important is to not place any constraints on the distance between the two reads. So what one could do is to align the first read (R1) on one side the second read (R2) on the other side and then merge the alignment to avoid any constraints placed by insert size.

[8:13](https://youtu.be/AFHaX_GRNBU?list=PL3xpfTVZLcNiSvvPWORbO32S1WDJqKp1e&t=493) Then, once you have the reads aligned on the genome, there’s a filtering step where we want to only keep what are called valid pairs, which represent the valid ligated product and we try and reconstruct the ligated product based on the alignment. The invalid read pairs are discarded: singletons, dangling ends, self-circle, and dumped pairs. Additional features on mapping quality, insert size, duplicates, restriction fragment size etc can be appended.

[9:22](https://youtu.be/AFHaX_GRNBU?list=PL3xpfTVZLcNiSvvPWORbO32S1WDJqKp1e&t=562) Once we have only the valid pairs, we can start constructing contact maps, which are not very complicated to do. We count how many times the two interactors A and B have been sequenced together as a proxy for contact probability. There is an additional step which is called normalisation which is done by a few algorithms called balancing. These can be applied to the HiC map. (Details in [Imakaev _et al_., 2012](https://www.nature.com/articles/nmeth.2148)).

[10:06](https://youtu.be/AFHaX_GRNBU?list=PL3xpfTVZLcNiSvvPWORbO32S1WDJqKp1e&t=606) Let’s address data formats. There are three main types of file formats for HiC data. The first is a .txt file, which is simple. HiC maps are symmetrical and we only therefore store half of the data. Then the 4G nucleome produced two other types of file formats: .hic files from the [Juicer suite](https://github.com/aidenlab/juicer) which can be put into Juicebox for the visualisation of HiC data, and .cool files that tends to be the standard format for HiC data because it is python oriented and there are many python-based packages that can be used for data analysis. These files are compatible with Higlass for visualisation.

[11:24](https://youtu.be/AFHaX_GRNBU?list=PL3xpfTVZLcNiSvvPWORbO32S1WDJqKp1e&t=684) So you have a contact map, and you’d like to look at TADs and loops and so on. As far as I know, there is no standard yet and the figure I show here is from [Forcato _et al_., 2017](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5493985/) and you see there are many different results for loop calling from the different available tools.

[12:11](https://youtu.be/AFHaX_GRNBU?list=PL3xpfTVZLcNiSvvPWORbO32S1WDJqKp1e&t=731) We started developing the nf-core/hic pipeline in 2019 and it’s currently at version 1.3. The idea behind this was to have a community-driven pipeline, to perform data processing, run quality control on HiC data compatible with all the various steps in the protocol, and finally to start moving into more downstream analysis.

[13:15](https://youtu.be/AFHaX_GRNBU?list=PL3xpfTVZLcNiSvvPWORbO32S1WDJqKp1e&t=795) The first version of the pipeline was a Nextflow-based implementation of the HiC-Pro pipeline that I developed several years ago, which is still highly used in the field. HiC-Pro starts from the FastQ file, performs read-mapping, detects valid interaction products, and constructs the maps.

[13:51](https://youtu.be/AFHaX_GRNBU?list=PL3xpfTVZLcNiSvvPWORbO32S1WDJqKp1e&t=831) From v1.3, we started to move a little bit and especially for the contact maps we are now using .cool files with the [Cooler package](https://github.com/open2c/cooler). There’s still the option of backward compatibility if you’d like to work with --hicpro_maps. We also started to implement tools for downstream analysis to call the TADs using two different methods: CoolTools and HiCexplorer. We also added another couple of QCs like distance decays and so on.

[14:38](https://youtu.be/AFHaX_GRNBU?list=PL3xpfTVZLcNiSvvPWORbO32S1WDJqKp1e&t=878) Running the pipeline is not too complicated. You provide the FastQ file and the reference genome. If the protocol involved a digestion step, you input the name of the restriction enzyme that was used. The pipeline automatically sets up the restriction site and the ligation site that should be used for the analysis. If the restriction enzyme you use is not in the list, you can also set up this option manually. We also have a mode for DNase HiC where you use DNase instead of a restriction enzyme and you use the option DNase. We use the minimum distance between two contacts as a filter.

[15:50](https://youtu.be/AFHaX_GRNBU?list=PL3xpfTVZLcNiSvvPWORbO32S1WDJqKp1e&t=950) This slide shows a set of more advanced commands. If I were to translate these, it’s basically similar to what I said before. We input the type file, provide the reference genome, the type of restriction enzyme used. Then the reads are split into chunks to be able to analyse all the chunks in parallel on the cluster. Then I ask the pipeline to generate the contact maps at different resolutions; these define the size of the window for the different contact maps. I provide a resolution size for the compartments. Then I state that I’d like to run two TADs callers: `insulation` and `hicexplorer` using a resolution of 40 kb.

[17:14](https://youtu.be/AFHaX_GRNBU?list=PL3xpfTVZLcNiSvvPWORbO32S1WDJqKp1e&t=1034) There are a few quality controls at the end of a pipeline. These are based on the `--hic-pro` module, and this gives a few graphs. The first maps statistics, which indicates how many reads align to the reference genome. The dark blue ones are fully aligned, and the lighter ones are where `--hic-pro` found ligation sites. The latter is of course expected, but overall the graph provides information on the quality of data. Then we also merge the .bam files for the paired-end reads and filter the data according to mapping quality and so on.

[18:19](https://youtu.be/AFHaX_GRNBU?list=PL3xpfTVZLcNiSvvPWORbO32S1WDJqKp1e&t=1099) Another means of quality control is the valid-pair detection. We expect to have a lot of valid pairs, much more than invalid pairs. This provides information on whether there were issues during ligation and the subsequent pull-down step so that you can modify your protocol in the lab accordingly. Finally, the last type of plot depicts a representation of the distance between the contacts. You see contacts in _cis_ i.e. within the chromosome, and the rest in _trans_ which represent contacts between chromosomes.

[19:08](https://youtu.be/AFHaX_GRNBU?list=PL3xpfTVZLcNiSvvPWORbO32S1WDJqKp1e&t=1148) So for the future. As it stands today, the most important thing is to port the pipeline to DSL2, which we hope to do this year. There are also several other things that need to be done, and you can see them here on the screen. You’re welcome to join us in developing this pipeline further. Thank you and you can contact me on Slack if you have any questions.

</details>
