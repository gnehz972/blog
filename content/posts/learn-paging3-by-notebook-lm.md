---
title: "Learn Paging3 by NotebookLM"
date: 2024-12-05T22:52:01+08:00
tags:
  - NotebookLM
  - Android Paging3
categories:
  - Android
---

The initial purpose is to understand how the prepend loading in Paging3 is triggered, is it able to manually triggered with flexibility, 
and it extended to get to know the whole lib well.

Although Paging3 is a complicate Android lib with a bunch of classes to do the paging stuff, the NotebookLM helps explain the concept well.
By pasting all the source code to it as context, it explains the concept and the relationship between core classes, and draws the diagram for me.

## Preparation
1. Download all the paging3 source code using https://downgit.github.io/#/home
2. Paging3 common lib source code: https://github.com/androidx/androidx/tree/androidx-main/paging/paging-common/src/commonMain/kotlin/androidx/paging
3. Use copy-files-to-clipboard-for-llm plugin to merge all the files and copy to clipboard
4. Paste the source code to NotebookLM as the reference source
5. Chat with NotebookLM to get explanation and diagrams

## Sequence Diagram

```mermaid
sequenceDiagram

participant UI

participant LazyPagingItems

participant PagingDataPresenter

participant HintReceiver

participant PageFetcher

participant PageFetcherSnapshot

participant PageFetcherSnapshotState

participant PagingSource

UI->>LazyPagingItems: Access item (get)

activate UI

activate LazyPagingItems

LazyPagingItems->>PagingDataPresenter: Get item at index (get)

activate PagingDataPresenter

PagingDataPresenter->>PagingDataPresenter: Get item from page store

PagingDataPresenter->>HintReceiver: Access Hint (ViewportHint)

deactivate UI

activate HintReceiver

HintReceiver->>PageFetcher: Access Hint (ViewportHint)

deactivate HintReceiver

activate PageFetcher

PageFetcher->>PageFetcherSnapshot: Access Hint (ViewportHint)

activate PageFetcherSnapshot

PageFetcherSnapshot->>PageFetcherSnapshot: Wrap hint with generationId (GenerationalViewportHint)

PageFetcherSnapshot->>PageFetcherSnapshotState: Get next load key (nextLoadKeyOrNull)

activate PageFetcherSnapshotState

PageFetcherSnapshotState-->>PageFetcherSnapshot: Return load key

deactivate PageFetcherSnapshotState

alt Valid load key and prefetch distance not fulfilled

PageFetcherSnapshot->>PagingSource: Load data (load, LoadParams{loadType=PREPEND, key, loadSize})

activate PagingSource

PagingSource->>PagingSource: Fetch data

PagingSource-->>PageFetcherSnapshot: Load result (LoadResult{Page/Error})

deactivate PagingSource

PageFetcherSnapshot->>PageFetcherSnapshotState: Insert page, update load state, check for drop (insert)

activate PageFetcherSnapshotState

PageFetcherSnapshotState-->>PageFetcherSnapshot: Confirmation

deactivate PageFetcherSnapshotState

else Load key invalid or prefetch distance fulfilled

PageFetcherSnapshot->>PageFetcherSnapshot: Skip load

end

deactivate PageFetcherSnapshot

deactivate PageFetcher

deactivate PagingDataPresenter

LazyPagingItems-->>UI: Return item

deactivate LazyPagingItems
```

## Core components

```mermaid
classDiagram

class UI

class LazyPagingItems {

-flow: Flow<PagingData<T>>

-pagingDataPresenter: PagingDataPresenter<T>

+refresh()

+loadState: CombinedLoadStates

+get(index: Int): T?

}

class PagingDataPresenter {

-hintReceiver: HintReceiver

-uiReceiver: UiReceiver

-pageStore: PageStore

+refresh()

+get(index: Int) : T?

+addLoadStateListener(listener)

+removeLoadStateListener(listener)

+presentPagingDataEvent(event: PagingDataEvent)

}

class HintReceiver {

+accessHint(hint: ViewportHint)

}

class PageFetcher {

-snapshot: PageFetcherSnapshot

+refresh()

}

class PageFetcherSnapshot {

-pagingSource: PagingSource

-stateHolder: PageFetcherSnapshotState.Holder

+doLoad(loadType: LoadType, hint: GenerationalViewportHint)

+doInitialLoad()

}

class PageFetcherSnapshotState {

-pages: List<TransformablePage<Key, Value>>

-placeholdersBefore: Int

-placeholdersAfter: Int

+insert(generationId: Int, loadType: LoadType, page: Page) : Boolean

+nextLoadKeyOrNull(loadType: LoadType, generationId: Int, presentedItemsBeyondAnchor: Int) : Key?

}

class PagingSource {

+load(params: LoadParams) : LoadResult

}

  

UI ..> LazyPagingItems

LazyPagingItems *-- PagingDataPresenter

PagingDataPresenter ..> HintReceiver

HintReceiver ..> PageFetcher

PageFetcher *-- PageFetcherSnapshot

PageFetcherSnapshot *-- PageFetcherSnapshotState

PageFetcherSnapshot ..> PagingSource
```
