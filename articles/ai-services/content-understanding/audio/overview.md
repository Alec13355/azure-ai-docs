---
title: Azure AI Content Understanding audio overview
titleSuffix: Azure AI services
description: Learn about Azure AI Content Understanding audio solutions
author: laujan
ms.author: lajanuar
manager: nitinme
ms.service: azure
ms.topic: overview
ms.date: 11/19/2024
---


# Azure AI Content Understanding audio solutions overview (preview)

> [!IMPORTANT]
>
> * Azure AI Content Understanding is available in preview. Public preview releases provide early access to features that are in active development.
> * Features, approaches, and processes may change or have constrained capabilities, prior to General Availability (GA).
> * For more information, *see* [**Supplemental Terms of Use for Microsoft Azure Previews**](https://azure.microsoft.com/support/legal/preview-supplemental-terms).

Content Understanding audio capabilities can help you transcribe and diarize conversational audio. The feature can also help you generate enhanced outputs like summaries, special industry record formats, captioning data. Content Understanding audio and audio capabilities enable you to extract valuable information such as key topics, sentiment, and more. To get started, use one of the provided out-of-box prebuilt extraction schemas and start generating results. You can also customize Content Understanding capabilities to meet your business needs as necessary.

Here are some of the common scenarios for Content Understanding extracted conversational audio data:

* Get customer insights through summarization and sentiment.

* Generate Call Center call analytics results.

* Create a redacted version of the transcript with personally identifiable information (`PII`) removed.

* Analyze recordings to find valuable information like most desired topics.

## Content Understanding in AI Studio

AI studio enables you to set up, test, and manage Content Understanding solutions. You can use prebuilt schemas that can be customized to analyze your audio transcripts to easily generate results matching your specific business needs. A typical scenario is to automatically process files uploaded into a blob storage account and write the analytics results back to it. Based on the single file analysis, you can then easily index and add these results to a database or an Azure AI Search Index to easily generate more cross-recording insights and dashboards.

* Get insights from audio recordings of meetings, calls, and conversations. Review insights from summaries, sentiment results, action items, meeting notes, and `PII` redacted transcripts.

* Customize the results according to your specific needs and scenarios to modify the output of the workflow.

* Test and deploy customized workflows easily and quickly, without having to write any code or use any external tools.

* Access and manage your Content Understanding projects and resources in one place, along with other AI services that you use in AI Studio.

 You can use the AI Studio UI to manage audio analytics projects and resources.

* You might prefer to use AI studio for Content Understanding because it offers a user-friendly interface and an easy-to-use setup experience to generate insights from audio data. You can also test and deploy different versions of the output schema directly in AI studio.

* Developers can use the SDK and APIs to process data at scale in production and to integrate Content Understanding into Azure Pipelines as needed.

## Content Understanding audio workflow

The following diagram provides a high-level overview of a typical Content Understanding Audio processing workflow.

  :::image type="content" source="../media/audio/overview/workflow-diagram.png" lightbox="../media/audio/overview/workflow-diagram.png" alt-text="Illustration of Content Understanding audio workflow.":::

A typical Content UnderstandingAudio workflow consists of the following steps:

1. You send audio or transcription files to the Content UnderstandingAPI wither as single file or providing settings to process from a connected blob storage account.

1. Content UnderstandingContent Extraction generates a conversation transcript incl. speaker separation in webVTT format and optionally recognizes speaker roles or names to replace generic 'Speaker n' results.

1. The Content UnderstandingField Extraction then generates added insights based on the generated conversation transcript.

1. The Content Understanding service returns an audio file results containing the conversation transcript including added generated insights in JSON format. The results are either directly returned from the API or can be written into a connected blob storage account.

## Content Understanding prebuilt audio scenarios

Content Understanding provides the following customizable prebuilt scenario templates:

* **Post call analytics**. Analyze call recordings and generate outputs such as conversation transcript, call summary, sentiment assessment and more.

* **Conversation summarization**. Generate transcriptions from conversation audio recordings, generate a summary, and assess sentiment.

 You can start with any prebuilt scenario or start from scratch to get started and customize as needed to meet your business needs.

## Audio format support and input requirements

For a complete list of  Content Understanding supported audio formats, *see* our [Service limits and codecs](../service-limits.md) page.

## Supported regions, languages, and locales

For a complete list of supported regions, languages, and locales, see our [Language and region support](../language-region-support.md)) page.


**Content Understanding audio capability operation limits**

|Attribute|Limit|
|-----|-----|
|Time|Maximum of 2 hours in length|
|Size|Maximum of 200 MB in size|
|Speakers|Maximum number of 36 speakers|



