# Model Documentation

## Criteria

### The car is able to drive at least 4.32 miles without incident..
>** I follow the Project walkthrough and Q&A Video to have the structure of the code. 
And have added my logic to change lanes and avoid collisions. Actually after testing , the car can have no issue for more than 40 mins.

![pic](https://github.com/bryan0806/Udacity-CarND-Path-Planning-Project/blob/master/pic/pic01.png)

### The car drives according to the speed limit.
>** Yes, here is my code for speed control:
1. To have avoid acceleration jerk, the start with 0 speed, then adding 0.224 mph for each update until we get to 49.5 which is speed limit.
2. If the car is too closed to front car, start to decrease the speed by 0.224 mph each update until we have same speed with front car.
3. If our car have very closed distance with suddenly changing lane car in front of the car, we further decrease more speed. 
```
if(too_closed)
                {
                    if(ref_vel>check_speed)
                    {
                        ref_vel -= .224;
                    }
                    
                }
                else if(ref_vel < 49.5)
                {
                    ref_vel += .224;
                }
            
                if(danger)
                {
                    ref_vel -= .224;
                    
                }
```

### Max Acceleration and Jerk are not Exceeded.
>** Like above explain, my car increase/decrease speed by 0.224 mph each update not to exceed max acceleration and jerk.
Further more, I also use advised spline function to make turning more smoothly to avoid jerk.

```
                //create a spline
                tk::spline s;

                //set (x,y) points to the spline
                s.set_points(ptsx,ptsy);


                // Start with aa of the previous path points from last time
                for(int i =0; i < previous_path_x.size();i++)
                {
                    next_x_vals.push_back(previous_path_x[i]);
                    next_y_vals.push_back(previous_path_y[i]);

                }

                //Calculate how to break up spline points so that we travel at our desired reference velocity
                double target_x = 30.0;
                double target_y = s(target_x);
                double target_dist = sqrt((target_x)*(target_x)+(target_y)*(target_y));

                double x_add_on = 0;

                //fill up the rest of our path planner after filling it with previous points, here we will always output 50 points

                for(int i = 1; i <= 50-previous_path_x.size();i++)
                {
                    double N = (target_dist/(.02*ref_vel/2.24));
                    double x_point = x_add_on+(target_x)/N;
                    double y_point = s(x_point);

                    x_add_on = x_point;

                    double x_ref = x_point;
                    double y_ref = y_point;

                    //rotate back to normal after rotating it earlier
                    x_point = (x_ref*cos(ref_yaw)-y_ref*sin(ref_yaw));
                    y_point = (x_ref*sin(ref_yaw)+y_ref*cos(ref_yaw));

                    x_point += ref_x;
                    y_point += ref_y;

                    next_x_vals.push_back(x_point);
                    next_y_vals.push_back(y_point);

                }
```

### Car does not have collisions.
### The car is able to change lanes
Here is my logic for collisions avoidance and also change lane:
1. First I will check if car is in front of our lane, and if other car is within the range of 30m.
```
if((check_car_s > car_s) && ((check_car_s-car_s) < 30))
                        {
                            //ref_vel = check_speed;
                            too_closed = true;
                            if((check_car_s > car_s) && ((check_car_s-car_s) < 15)) // prevent front car suddenly change lane
                            {
                                danger = true;
                            }
```
                            
2. Then I would check left/right lane if there is car near us in some range.

```
// check if there is car in the left lane and safe to turn
                            if(too_closed && lane==1)
                            {
                                
                                bool ready_to_swift_left = false;
                                bool ready_to_swift_right = false;
                                int safe_count_left = 0;
                                int safe_count_right = 0;

                                
                                for(int j = 0; j < sensor_fusion.size();j++)
                                {
                                    float d = sensor_fusion[j][6];
                                    
                                    
                                    
                                    if(d < (2+4*(lane-1)+2) && d > (2+4*(lane-1)-2)  ) // left lane has car and
                                    {
                                        cout << "yes, there is car in the left" << endl;
                                        cout << "my car s is " << car_s << endl;
                                        double check_car_s_left = sensor_fusion[j][5];
                                        cout << "the car id :" << j << " s:" << check_car_s_left << endl;
                                        if((check_car_s_left-car_s > 50 or car_s - check_car_s_left > 30) ){ // left lane is safe to swift
                                            cout << "ready to swift left !!!!!" << endl;
                                            //lane = 0 ; // to prevent there could still car not yet checking , can not swift now
                                            ready_to_swift_left = true;
                                         }
                                        else if(fabs(check_car_s_left-car_s) < 30){
                                            safe_count_left += 1;
                                        }
                                        
                                        
                                    }
                                    
                                    else if(d < (2+4*(lane+1)+2) && d > (2+4*(lane+1)-2) ) // check right lane
                                    {
                                        cout << "yes, there is car in the right" << endl;
                                        cout << "my car s is " << car_s << endl;
                                        double check_car_s_right = sensor_fusion[j][5];
                                        cout << "right car id :" << j << " s:" << check_car_s_right << endl;
                                        if((check_car_s_right-car_s > 50 or car_s - check_car_s_right > 30) ){ // right lane is safe to swift
                                            cout << "ready to swift to right!!!!!" << endl;
                                            //lane =  2; / to prevent there could still car not yet checking , can not swift now
                                            ready_to_swift_right = true;
                                        }
                                        else if(fabs(check_car_s_right-car_s) < 30){
                                            safe_count_right += 1;
                                        }
                                        
                                    }
                                }
```
                                

3.Then, after check all other cars , then decide which lane to change

```

                            
                            
                                
                                if(ready_to_swift_left==true and safe_count_left <= 0){
                                    lane = 0;
                                }else if(ready_to_swift_right==true and safe_count_right <= 0){
                                    lane = 2;
                                }
```

4. Same logic for lane = 0 or 2 


### The car stays in its lane, except for the time between changing lanes

>** Yes, I believe it is.

