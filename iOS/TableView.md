# TableView

### **테이블 뷰에 Custom Cell 디자인해서 화면에 출력하기까지 과정을 설명해주세요.**🌟

**Custom Cell 디자인**

스토리보드를 이용한다고 했을때, Xcode를 이용하여 Cell을 디자인할수 있습니다. 이때 **프로토타입 Cell**이 제공되는데, 셀을 디자인하기 위한 template역할을 합니다.

기본적으로 제공되는 스타일이 아닌 cell을 이용하고 싶으면 **UITableViewCell를 subclassing**하는 class를 만들고, outlet으로 연결해줘야합니다.

해당 Cell을 tableview가 이용하기위해서는 테이블 뷰의 tableView(_:cellForRowAt:) 메소드에서 **dequeueReusableCell(withIdentifier:for:)**를 이용해야합니다. 그래서 스토리보드에서 cell의 **Identifier를 꼭 지정해** 줘야합니다. 그리고 앞서 연결한 outlet에 값을 지정해주면됩니다.

**만약 스토리보드를 사용안한다면?**

위의 흐름과 같은데 dequeueReusableCell하기 이전에 cell **register** 단계를 거쳐야합니다. 

**화면에 출력하기까지**

화면에 tableview가 생성되고 처음으로 안에 있는 cell들이 보여질때는 **새롭운 객체가 생성됩니다**.  (함수 호출 순서는 **awakeFromNib** → tableView(_:cellForRowAt:) → tableView(_:willDisplay:forRowAt) )  

이후에 테이블 뷰를 스크롤하게되면 테이블뷰의 **queue에서 cell을 가져와서** 재활용하게됩니다. (그래서 reusable). 이때 Cell의 prepareForReuse함수를 호출하게됩니다.

가끔 reusable cell을 이용하다보면 다른 cell의 데이터가 남아있는 경우가 있는데 **prepareForReuse에서 cell을 다시 리셋해주는 작업을** 해줘야합니다.

**prefetch는 해봤나요?**

테이블뷰의 delegate와 datasource를 self로 지정해줬듯이 **prefetchDataSource를 self로 지정해줘야합니다**.

일반적으로 용량이 큰 **데이터를 미리 받아오기 위한 작업이 비동기적으로** 이루어집니다.

스크롤 방향이 반대로 바뀔때 tableView(_:cancelPrefetchingForRowsAt:)를 이용하여 **작업을 취소 할수도 있습니다.**

- 참고

    정말 간단한 코드 스니핏

    ```swift
    class MyCell: UITableViewCell {
        @IBOutlet weak var titleLabel: UILabel!
        
        override func awakeFromNib() {
            super.awakeFromNib()
            print("awakeFromNib")
        }
        
        override func prepareForReuse() {
            super.prepareForReuse()
            titleLabel.textColor = .label
            print("prepareForReuse")
        }
        
        @IBAction func didPressButton(_ sender: UIButton) {
            titleLabel.textColor  = .red
        }
        
    }
    ```

    ```swift
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
            let cell = tableView.dequeueReusableCell(withIdentifier: "MyCell", for: indexPath) as! MyCell
            cell.titleLabel.text = "cell #\(indexPath.row)"
            return cell
            
        }
    ```

    **cell이 재사용되는 과정**

    ![TableView%20bd5d128c3e034ae3aedace6ac53412ed/Untitled.png](TableView%20bd5d128c3e034ae3aedace6ac53412ed/Untitled.png)

    [https://jinnify.tistory.com/58](https://jinnify.tistory.com/58)

    [https://fluffy.es/solve-duplicated-cells/](https://fluffy.es/solve-duplicated-cells/)

    [https://developer.apple.com/documentation/uikit/views_and_controls/table_views/configuring_the_cells_for_your_table](https://developer.apple.com/documentation/uikit/views_and_controls/table_views/configuring_the_cells_for_your_table)

### **DiffableDataSource에 대해서 설명해주세요**🌟

**DiffableDataSource**는 **snapshot**을 이용해서 collection view 혹은 table view에게 **데이터를 제공**합니다.

특징으로는 **새로운 snapshot**이 apply되면 이전에 있던 snapshot과 비교하여 데이터 삭제/추가/재정렬 등을 자동으로 적용합니다.

**snapshot이 뭔가요?**

현재 데이터의 state를 나타냅니다.

snapshot은 section과 item으로 구성되어있으며, 각각 Hashable protocol을 채택해야합니다. 일반적으로 UUID를 id값으로 줍니다. section과 같은 경우에는 enum을 쓰는경우도 있습니다.

**DiffableDataSource를 썼을 때 장점이 뭔가요**

크게 두가지 이점이 있는데, 애니메이션이 자동으로 적용된다는 점, 그리고 안전하다는점이 있습니다.

첫번째는 데이터가 바뀔때 화면 변화 애니메이션이 자동적으로 적용됩니다. 이전에는 데이터에서 변화가 생기면 reloadData()을 호출해줘야했는데, 이때는 애니메이션이 없습니다.

두번째로는 더 안전하다는것입니다. 이전에는 performBatchUpdates() 를 이용할때 뷰와 데이터간의 동기화가 제대로 안되어있으면 오류가 났습니다. DiffableDataSource는 source of truth 가 snapshot 하나기이때문에 동기화에 대해서 따로 신경쓰지 않아도 됩니다.

- 참고

    ```swift
    typealias DataSource = UICollectionViewDiffableDataSource<Section, Video>
    typealias Snapshot = NSDiffableDataSourceSnapshot<Section, Video>

    func makeDataSource() -> DataSource {
        // 1
        let dataSource = DataSource(
          collectionView: collectionView,
          cellProvider: { (collectionView, indexPath, video) ->
            UICollectionViewCell? in
            // 2
            let cell = collectionView.dequeueReusableCell(
              withReuseIdentifier: "VideoCollectionViewCell",
              for: indexPath) as? VideoCollectionViewCell
            cell?.video = video
            return cell
        })
    }

    func applySnapshot(animatingDifferences: Bool = true) {
        var snapshot = Snapshot()
        snapshot.appendSections(sections)
        sections.forEach { section in
          snapshot.appendItems(section.videos, toSection: section)
        }
        dataSource.apply(snapshot, animatingDifferences: animatingDifferences)
    }
    ```

    [https://developer.apple.com/documentation/uikit/nsdiffabledatasourcesnapshot](https://developer.apple.com/documentation/uikit/nsdiffabledatasourcesnapshot)

    `nsdiffabledatasourcesnapshot` : A representation of the state of the data in a view at a specific point in time.

    [https://developer.apple.com/documentation/uikit/uitableviewdiffabledatasource](https://developer.apple.com/documentation/uikit/uitableviewdiffabledatasource)

    - Diffable data sources **use snapshots to provide data** for collection views and table views.
    - It also conforms to the UITableViewDataSource protocol and provides implementations for all of the protocol’s methods.

    [https://www.donnywals.com/modern-table-views-with-diffable-data-sources/](https://www.donnywals.com/modern-table-views-with-diffable-data-sources/)

    [https://www.avanderlee.com/swift/diffable-data-sources-adoption/](https://www.avanderlee.com/swift/diffable-data-sources-adoption/)

    [https://medium.com/better-programming/applying-diffable-data-sources-70ce65b368e4](https://medium.com/better-programming/applying-diffable-data-sources-70ce65b368e4)

    For updating, both the approaches, `reloadData()` and `performBatchUpdates()`, had their own sets of issues.

    While `reloadData()` would ruin our chances of showing nice animations, `performBatchUpdates()` would easily lead to errors if not handled carefully.

    Errors like the one shown below were pretty common with `performBatchUpdates()`:

    ```
    Terminating app due to uncaught exception ‘NSInternalInconsistencyException’,
    reason: ‘Invalid update: invalid number of items in section 0.
    The number of items contained in an existing section after the update 
    must be equal to the number of items contained in that section before.
    ```

    Gladly, Apple brought in diffable data sources to tackle such errors on our behalf.