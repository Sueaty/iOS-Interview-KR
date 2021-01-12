### CoreData Migration

### Q.코어데이터를 사용할 때 어려움이 있었다고 했는데 어트리뷰트 타입이 변경될 경우 이전 버전과 매핑이 필요한 이유는 무엇일까요?

https://developer.apple.com/documentation/coredata

코어데이터는 앱의 데이터 모델의 최신화를 유지하기 위해서 migration tool을 제공합니다.

만약 migration 작업을 수행하지 않으면, 앱의 데이터 모델의 변경이 있었을 때 기존 앱을 사용하던 사용자는 기존 persistant container에 저장된 모델 데이터와 새로운 데이터 간의  충돌이 일어나게 되고 앱이 크래시 납니다.

따라서 꼭 migration 작업이 필요합니다.

migration에는 크게 두가지가 있는데, 데이터 모델의 변경 정도가 복잡하지 않아 자동적으로 migration이 가능한 lightweight migration과 lightweight migration의 범주를 벗어나 직접 mapping을 시켜줘야 하는 heavyweight migration이 있습니다.

lightweight migration 같은 경우는

1. 새로운 엔티티, 애트리뷰트, 릴레이션십이 추가되거나

2. 기존 엔티티, 애트리뷰트, 릴레이션십이 제거되거나

3. 논옵셔널 타입을 옵셔널 타입으로 변경하거나

등의 상황에서 사용 가능 합니다

저희 팀의 경우는 Int 타입이었던 애트리뷰트를 string 타입으로 변경해야 했기 때문에

`NSEntityMigrationPolicy` 을 채택한 custom migrationPolicy 객체를 만들어 직접 mapping 작업을 진행해 주었습니다.

```swift
class POITransformationPolicy: NSEntityMigrationPolicy {
    
    override func createDestinationInstances(forSource sInstance: NSManagedObject, in mapping: NSEntityMapping, manager: NSMigrationManager) throws {

        let POI = NSEntityDescription.insertNewObject(forEntityName: "POI", into: manager.destinationContext)
        
        let id = sInstance.value(forKey: "id") as? Int64 ?? 0
        let stringId = String(id)
        
        POI.setValue(stringId, forKey: "id")
        POI.setValue(sInstance.value(forKey: "lat"), forKey: "lat")
        POI.setValue(sInstance.value(forKey: "lng"), forKey: "lng")
 
        manager.associate(sourceInstance: sInstance, withDestinationInstance: POI, for: mapping)
    }
    
}
```

