# Week 4. Q1

public class Equipment : MonoBehaviour
{
    public Equip curEquip;
    public Transform equipParent;

    private PlayerController controller;
    private PlayerCondition condition;

    void Start()
    {
        controller = CharacterManager.Instance.Player.controller; 
        캐릭터 매니저에서 플레이어 인스턴스 컨트롤러에 대입
        condition = CharacterManager.Instance.Player.condition; 
        캐릭터 매니저에서 플레이어 인스턴스 컨디션에 대입
    }

    public void OnAttackInput(InputAction.CallbackContext context)
    {
        if(context.phase == InputActionPhase.Performed && curEquip != null && controller.canLook) 
        { 컨트롤러 볼 수 있고 장착한 아이템이 있으며 인풋이 같을 때
            curEquip.OnAttackInput();
        }  공격 함수 실행
    }

    public void EquipNew(ItemData data)
    {
        UnEquip();  장착해제함수
        curEquip = Instantiate(data.equipPrefab, equipParent).GetComponent<Equip>(); 
    }   데이터의 장착 프리팹 인스턴스하고 equip 컴포넌트를 가져온것을 curEquip에 대입

    public void UnEquip()
    {
        if(curEquip != null)  현재장착한 것이 널이 아니라면
        {
            Destroy(curEquip.gameObject);  장착한 게임오브젝트 파괴
            curEquip = null;   curEquip 변수 널로 
        }
    }
}



public class EquipTool : Equip  Equip 상속
{ 
    public float attackRate;
    private bool attacking;
    public float attackDistance;

    [Header("Resource Gathering")]
    public bool doesGatherResources;

    [Header("Combat")]
    public bool doesDealDamage;
    public int damage;

    private Animator animator;
    private Camera camera;

    private void Awake()
    {
        camera = Camera.main;  
        animator = GetComponent<Animator>(); 
    }

    public override void OnAttackInput()  오버라이드 덮어쓰기
    {
        if (!attacking)   공격상태가 아니라면
        {  
          attacking = true;  공격상태를 트루로
          animator.SetTrigger("Attack");  에니메이터의 트리거 어택
	        Invoke("OnCanAttack", attackRate);  OnCanAttack을 attackRate시간마다 실행
        }
    }

    void OnCanAttack()
    {
        attacking = false;  공격상태 false로
    }


public class Resource : MonoBehaviour
{
	public ItemData itemToGive;
	public int quantityPerHit = 1;
	public int capacity;

	public void Gather(Vector3 hitPoint, Vector3 hitNormal)
	{
		for(int i = 0; i < quantityPerHit; i++)  반복문
		{
			if (capacity <= 0) break;   양이 0보다 작거나 같다면 취소 / 방어식

			capacity -= 1;  양에 -1
			Instantiate(itemToGive.dropPrefab, hitPoint + Vector3.up, Quaternion.LookRotation(hitNormal, Vector3.up));  
		}           줘야하는 아이템의 드랍프리펩, 때린 지점의 위쪽, 위쪽방향으로 인스턴스화

		if(capacity <= 0)
		{
			Destroy(gameObject);  양이 0보다 작거나 같으면 파괴
		}
	}