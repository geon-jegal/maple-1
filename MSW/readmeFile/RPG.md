--[[
playerQuest에서 OnBeginPlay ---- started가 finished로 바뀌어야 되는 것 아닌가?
playerinventory에서 findslot와 additems
MonsterHit컴포넌트의 dead()함수
]]

-- 전체적으로 데이터 셋에 대부분의 데이터가 있고 데이터를 들고와서 처리, 출력하는 로직으로 구성 되어 있음
-- 동일한 이미지들도 사용 유형에 따라 여러가지 ruid가 있다.
-- 몬스터ai컴포넌트 attackBall = 투사체
-- hit, attack컴포넌트들은 모두 오버라이드 함

--[[
UserDataSetLogic은 데이터 셋을 가져오고 설정(추가, 삭제)하는 로직(몬스터, 아이템, npc, 퀘스트, 맵, 포탈)
ResourceLogic은 데이터 셋에서 데이터를 들고오는 로직
SystemMessageLogic 메세지 출력, 삭제 로직
PlayerLogic 레벨, 스텟, 장비에 의한 플레이어 스텟 변화를 계산하는 로직 -> 레벨 업으로 인한 스텟 증가만 있고 장비 착용으로 인한 스텟 증가는 없음 -- 빈약
MapLogic, CanvasLogic, MapPoint, MapLine 맵을 만들고 생성하는 로직
LocaleLogic 데이터 셋의 언어 타입 들고오기
ItemLogic 아이템의 정보를 받아와 아이템 생성 시 아이템의 정보를 성정하고 아이템 습득 시 인벤토리에 넣거나 아이템을 삭제하는 로직 
-> 아이템 생성, 삭제, 인벤토리에 넣기, 메소 습득 시 플레이어가 가지고 있는 메소 업데이트를 하는 로직
IDLogic 상황에 맞는 스프라이트를 사용하기 위한 로직
CanvasLogic 월드 맵에 사용하는 포인트롸 선을 복사, 제거하는 로직
InventoryLogic 변수 : currentSlot, draggingSlot
 
----------------------------------------------------------------------------------------------------------

PlayerData 서버에서 플레이어 데이터를 들고와 게임 시작 시 설정, 게임 종료 시 플레이어의 데이터를 서버롤 넘겨 저장하는 컴포넌트
PlayerEquipment 플레이어가 장비를 착용하고, 헤제하는 컴포넌트 -- 빈약
PlayerInfo 플레이어 스텟을 설정하는 컴포넌트
PlayerLogic 레벨, 스텟, 장비에 의한 플레이어 스텟 변화를 계산하는 로직 -> 레벨 업으로 인한 스텟 증가만 있고 장비 착용으로 인한 스텟 증가는 없음 -- 빈약
PlayerUI 플레이어 스텟을 ui에 맵 이동 또는 실시간으로 업데이트 하는 함수 + 처음 게임에 입장 시 돌리는 주사위도 구현
PlayerQuest 플레이어가 시작한 퀘스트, 완료한 퀘스트를 서버에서 들고와 ui에 seting한다, 또 퀘스트ui자체를 구현한 컴포넌트
-- 퀘스트ui 구현과 퀘스트 클리어 로직이 빈약
PlayerInventory 플레이어의 인벤토리, 아이템 관리부터 상점 관련 기능들도 전부 여기에 있다 -- 빈약
PlayerAttack 데이터 셋에서 스킬의 데이터를 들고와 데이터에 맞게 출력한다, 
데미지 공식은 PlayerLogic:GetDamage() 함수에서 결정하고 그 스공에 스킬 데미지를 곱해야 하지만 이곳에는 스킬 데미지가 따로 없다.
PlayerHit 피격 시 플레이어 이미지의 깜빡거림, 넉백, 피격 시 무적을 작동하고 플레이어의 상태에 따라 히트박스도 결정한다

-----------------------------------------------------------------------------------------------------------

MonsterInfo 몬스터 생성 시 몬스터의 데미지나 처치 시 떨어지는 재화 설정 컴포넌트
MonsterAttack 데이터 셋에서 몬스터 공격시 데미지를 들고와 조건(자신이 사망한 상태인지, 피격자가 플레이어인지, 공격 범위 안에 있는지 등)을 판단 후 공격한다
MonsterHit 피격과 사망, 사망 시 드랍하는 아이템과 그 확률, 리스몬을 계산하는 컴포넌트
MonsterStateAnimation StateAnimation컴포넌트를 오버라이딩 한 것으로 현재 몬스터의 상태를 문자열로 리턴 한다.
MonsterAI 몬스터의 상태를 추가적으로 설정하고, 실시간 상태에 따른 행동을 결정하는 컴포넌트
StaticMonsterAI 고정되어 있거나 날아다니는(지형을 무시하는) 몬스터의 컴포넌트

-----------------------------------------------------------------------------------------------------------

Ball 투사체에 관련된 컴포넌트, 투사체의 출돌 범위나 데미지, 이동을 설정하고 일정 시간이 지나면 사라지거나 플레이어 피격을 구현한다
NPC npc 관련 컴포넌트, npc유형(상점, 일반 대화, 퀘스트)에 따른 행동을 정한다.
-> 자신(npc)이 플레이어에게 요청해야하는 퀘스트나 완료해야하는 퀘스트를 찾아 출력한다. 또 출력해야하는 ui가있다면 출력한다.
MapLogic, CanvasLogic, MapPoint, MapLine 맵을 만들고 생성하는 로직, ui도 정한다.
Portal 포털을 설정하는 컴포넌트
Item 아이템 습득에 관한 로직

-----------------------------------------------------------------------------------------------------------

CloseButton 부모 엔티티를 비활성화 시키는 버튼
DragButton UI를 잡고 움직이게 할 수 있게(드래그기능) 하는 컴포넌트
BuyButton 아이템 구매 버튼 컴포넌트(구매 버튼이 따로 구현 되어 있다. 부모 엔티티에 해당하는 ruid에 해당하는 아이템을 플레이어 인벤토리에 추가) 
InventorySlot 아이템 홀드 구현
CellButton 판매 구현 컴포넌트
]]