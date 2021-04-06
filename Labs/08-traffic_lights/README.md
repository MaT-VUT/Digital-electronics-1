# Lab assignment



## 1 Preparation tasks

Completed state table

| INPUT P | CLOCK | STATE | OUTPUT R |
| :-----: | :---: | :---: | :------: |
|    0    |   ↑   |   A   |    0     |
|    0    |   ↑   |   A   |    0     |
|    1    |   ↑   |   B   |    0     |
|    1    |   ↑   |   C   |    0     |
|    0    |   ↑   |   C   |    0     |
|    1    |   ↑   |   D   |    1     |
|    0    |   ↑   |   A   |    0     |
|    1    |   ↑   |   B   |    0     |
|    1    |   ↑   |   C   |    0     |
|    1    |   ↑   |   D   |    1     |
|    1    |   ↑   |   B   |    0     |
|    0    |   ↑   |   B   |    0     |
|    0    |   ↑   |   B   |    0     |
|    1    |   ↑   |   C   |    0     |
|    1    |   ↑   |   D   |    1     |
|    1    |   ↑   |   B   |    0     |

Figure with connection of RGB LEDs on Nexys A7 board and completed table with color settings

| RGB  |      PIN      |  RED  | YELLOW | GREEN |
| :--: | :-----------: | :---: | :----: | :---: |
| LD16 | N15, M16, R12 | 1,0,0 | 1,1,0  | 0,1,0 |
| LD17 | N16, R11, G14 | 1,0,0 | 1,1,0  | 0,1,0 |

![](images/2.jpg)



## 2 Traffic light controller

State diagram

Listing of VHDL code of sequential process `p_traffic_fsm`

```vhdl
    p_traffic_fsm : process(clk)
    begin
        if rising_edge(clk) then
            if (reset = '1') then       -- Synchronous reset
                s_state <= STOP1 ;      -- Set initial state
                s_cnt   <= c_ZERO;      -- Clear all bits

            elsif (s_en = '1') then
                -- Every 250 ms, CASE checks the value of the s_state 
                -- variable and changes to the next state according 
                -- to the delay value.
                case s_state is

                    -- If the current state is STOP1, then wait 1 sec
                    -- and move to the next GO_WAIT state.
                    when STOP1 =>
                        -- Count up to c_DELAY_1SEC
                        if (s_cnt < c_DELAY_1SEC) then
                            s_cnt <= s_cnt + 1;
                        else
                            -- Move to the next state
                            s_state <= WEST_GO;
                            -- Reset local counter value
                            s_cnt   <= c_ZERO;
                        end if;

                    when WEST_GO =>
                        if (s_cnt < c_DELAY_4SEC) then
                            s_cnt <=s_cnt + 1;
                        else
                            s_state <= WEST_WAIT;
                            s_cnt   <= c_ZERO;
                        end if;
                        
                    when WEST_WAIT =>
                        if (s_cnt < c_DELAY_2SEC) then
                            s_cnt <= s_cnt + 1;
                        else
                            s_state <= STOP2;
                            s_cnt   <= c_ZERO;
                        end if;
                        
                    when STOP2 =>
                        if (s_cnt < c_DELAY_1SEC) then
                            s_cnt <= s_cnt + 1;
                        else
                            s_state <= SOUTH_GO;
                            s_cnt   <= c_ZERO;
                        end if;
                        
                    when SOUTH_GO =>
                        if (s_cnt < c_DELAY_4SEC) then
                            s_cnt <= s_cnt + 1;
                        else
                            s_state <= SOUTH_WAIT;
                            s_cnt   <= c_ZERO;
                        end if;
                        
                    when SOUTH_WAIT =>
                        if (s_cnt < c_DELAY_2SEC) then
                            s_cnt <= s_cnt + 1;
                        else
                            s_state <= STOP1;
                            s_cnt   <= c_ZERO;
                        end if;


                    -- It is a good programming practice to use the 
                    -- OTHERS clause, even if all CASE choices have 
                    -- been made. 
                    when others =>
                        s_state <= STOP1;

                end case;
            end if; -- Synchronous reset
        end if; -- Rising edge
    end process p_traffic_fsm;
```
Listing of VHDL code of combinatorial process `p_output_fsm`

```vhdl
p_output_fsm : process(s_state)
    begin
        case s_state is
            when STOP1 =>
                south_o <= "100";   -- Red (RGB = 100)
                west_o  <= "100";   -- Red (RGB = 100)
                
            when WEST_GO =>
                south_o <= "100";   -- Red (RGB = 100)
                west_o  <= "010";   -- Red (RGB = 010)
                
            when WEST_WAIT =>
                south_o <= "100";   -- Red (RGB = 100)
                west_o  <= "110";   -- Red (RGB = 011)
            
            when STOP2 =>
                south_o <= "100";   -- Red (RGB = 100)
                west_o  <= "100";   -- Red (RGB = 100)
                
            when SOUTH_GO =>
                south_o <= "010";   -- Red (RGB = 010)
                west_o  <= "100";   -- Red (RGB = 100)
                
            when SOUTH_WAIT =>
                south_o <= "110";   -- Red (RGB = 011)
                west_o  <= "100";   -- Red (RGB = 100)                

            when others =>
                south_o <= "100";   -- Red
                west_o  <= "100";   -- Red
        end case;
    end process p_output_fsm;
```
Screenshot(s) of the simulation, from which it is clear that controller works correctly

![](images/1.jpg)

## 3 Flip-flops

State table

|   STATE    |  NO CARS   | CARS FROM EAST | CARS FROM WEST | CARS FROM EAST, WEST |
| :--------: | :--------: | :------------: | :------------: | :------------------: |
|   STOP1    |  WESTO_GO  |    SOUTH_GO    |    WEST_GO     |       WEST_GO        |
|  WEST_GO   | WEST_WAIT  |   WEST_WAIT    |    WEST_GO     |       WEST_GO        |
| WEST_WAIT  |   STOP2    |     STOP2      |     STOP2      |        STOP2         |
|   STOP2    |  SOUTH_GO  |    SOUTH_GO    |    WEST_GO     |       SOUTH_GO       |
|  SOUTH_GO  | SOUTH_WAIT |    SOUTH_GO    |   SOUTH_WAIT   |       SOUTH_GO       |
| SOUTH_WAIT |   STOP1    |     STOP1      |     STOP1      |        STOP1         |

State diagram

![](images/3.jpg)



Listing of VHDL code of sequential process `p_smart_traffic_fsm`

```vhdl
        p_smart_traffic_fsm : process(clk, east, west)
    begin
        if rising_edge(clk) then
            if (reset = '1') then       -- Synchronous reset
                s_state <= STOP1 ;      -- Set initial state
                s_cnt   <= c_ZERO;      -- Clear all bits

            elsif (s_en = '1') then
                -- Every 250 ms, CASE checks the value of the s_state 
                -- variable and changes to the next state according 
                -- to the delay value.
                case s_state is

                    -- If the current state is STOP1, then wait 1 sec
                    -- and move to the next GO_WAIT state.
                    when STOP1 =>
                        -- Count up to c_DELAY_1SEC
                        if (s_cnt < c_DELAY_1SEC) then
                            s_cnt <= s_cnt + 1;
                        elsif (west = '1') then
                            -- Move to the next state
                            s_state <= WEST_GO;
                            -- Reset local counter value
                            s_cnt   <= c_ZERO;
                        elsif (west = '0') and (east = '1') then
                            s_state <= SOUTH_GO;
                            -- Reset local counter value
                            s_cnt   <= c_ZERO;                            
                        else
                            -- Move to the next state
                            s_state <= WEST_GO;
                            -- Reset local counter value
                            s_cnt   <= c_ZERO;                            
                        end if;

                    when WEST_GO =>

                        if (s_cnt < c_DELAY_4SEC) then
                            s_cnt <= s_cnt + 1;
                        elsif (west = '1') then
                            s_cnt <= s_cnt;    
                            
                        else
                            s_state <= WEST_WAIT;
                            s_cnt   <= c_ZERO;
                        end if;

                    when WEST_WAIT =>

                        if (s_cnt < c_DELAY_2SEC) then
                            s_cnt <= s_cnt + 1;
                        else
                            s_state <= STOP2;
                            s_cnt   <= c_ZERO;
                        end if;
                        
                    when STOP2 =>

                        if (s_cnt < c_DELAY_1SEC) then
                            s_cnt <= s_cnt + 1;
                        elsif (east = '1') then
                            s_state <= SOUTH_GO;
                            s_cnt   <= c_ZERO;
                        elsif (west = '1') and (east = '0') then
                            s_state <= WEST_GO;
                            s_cnt   <= c_ZERO;                            
                        else
                            s_state <= SOUTH_GO;
                            s_cnt   <= c_ZERO;
                        end if;

                    when SOUTH_GO =>

                        if (s_cnt < c_DELAY_4SEC) then
                            s_cnt <= s_cnt + 1;
                        elsif (east = '1') then
                            s_cnt <= s_cnt;    
                        
                        else
                            s_state <= SOUTH_WAIT;
                            s_cnt   <= c_ZERO;
                        end if;
                        
                    when SOUTH_WAIT =>

                        if (s_cnt < c_DELAY_2SEC) then
                            s_cnt <= s_cnt + 1;
                        else
                            s_state <= STOP1;
                            s_cnt   <= c_ZERO;
                        end if;                        
                        
      
                    -- It is a good programming practice to use the 
                    -- OTHERS clause, even if all CASE choices have 
                    -- been made. 
                    when others =>
                        s_state <= STOP1;

                end case;
            end if; -- Synchronous reset
        end if; -- Rising edge
    end process p_smart_traffic_fsm;
```
