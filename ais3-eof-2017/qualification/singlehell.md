## singlehell (250)
### solution
先用strings看程式，發現經過upx加殼，因此先以`upx -d`脫殼，接著以IDA進行反編譯，稍微trace一下可以看到接收回應的部分
```c
__int64 __fastcall get_response(__int64 response)
{
  __int64 result; // rax
  void *reward; // rax
  unsigned int player_damage; // ebp
  __int64 monster_hp; // rbx

  result = decode4(response);
  if ( (_DWORD)result == 2 )
  {
    player_damage = decode4(response);
    monster_hp = decode8(response);
    PlayerHP -= player_damage;
    result = __printf_chk(1LL, "[Info] Player is Damaged: %d\n", player_damage);
    MonsterHP = monster_hp;
  }
  else if ( (_DWORD)result == 7 )
  {
    puts("[Info] Congratulation !");
    reward = decode_str(response);
    result = __printf_chk(1LL, "[Info] Here is your reward : %s\n", reward);
  }
  return result;
}
```
這邊會接收server的回應，若是2則減少player的血量，並設定monster的血量，若是7則顯示reward。
由於monster的血量是根據server送來的回應設定的，因此應該無法修改，但是player是用扣的，因此試著將player扣血的部分
```
400ecb:       29 2d ef 11 20 00       sub    DWORD PTR [rip+0x2011ef],ebp
```
全部patch成nop，測試後成功，player無論如何都不會受傷，server也沒有進行檢查。

但是由於怪物血量太高，傷害還是太小，因此接著試著看看攻擊的部分：
```c
bool Attack()
{
  ...
  srand(time(0LL));
  rand_num = random();
  v7 = 8;
  v8 = 1;
  i = 0LL;
  v3 = rand_num
     - 10
     * (((signed __int64)((unsigned __int128)(0x6666666666666667LL * (signed __int128)rand_num) >> 64) >> 2)
      - (rand_num >> 63));
  v9 = rand_num
     - 10
     * (((signed __int64)((unsigned __int128)(0x6666666666666667LL * (signed __int128)rand_num) >> 64) >> 2)
      - (rand_num >> 63));
  do
  {
    v4 = (char *)&v8 + i++;
    *v4 ^= xor_code2[i];
  }
  while ( i != 8 );
  bytes_sent = send(fd, &v7, 0xCuLL, 0);
  if ( bytes_sent > 0 )
    __printf_chk(1LL, "[Info] Attack Monster with Damage: %d\n", v3);
  return bytes_sent > 0;
}
```
攻擊的傷害是隨機的，因此應該和`rand_num`有關，在設定`rand_num`的部分
```
  400c94:       e8 d7 fb ff ff          call   400870 <random@plt>
  400c99:       48 ba 67 66 66 66 66    movabs rdx,0x6666666666666667
  400ca0:       66 66 66
  400ca3:       48 89 c1                mov    rcx,rax
  400ca6:       48 8d 7c 24 04          lea    rdi,[rsp+0x4]
  400cab:       48 f7 ea                imul   rdx
  400cae:       48 89 c8                mov    rax,rcx
```
可以觀察到rax會複製給rcx再複製回rax，利用這點，試著將`mov rcx,rax`patch為`mov rcx,rdx`，這樣`rand_num`就會變為`0x6666666666666667`，實際測試後發現傷害增加非常多，因此以簡單的script不斷送攻擊請求等數十秒即可拿到flag了。

因為monster死後server似乎不會結束，所以最後會額外多幾個loop，導致要往回找一些才能看到flag。
